<script lang="ts">
    import { onMount } from 'svelte';
    // @ts-expect-error — warpjs ships no types
    import Warp from 'warpjs';

    type Mode = 'wave' | 'jelly' | 'tide' | 'none';

    type Props = {
        svg: string;
        mode?: Mode;
        fit?: 'natural' | 'stretch';
        complexity?: number;
        interpolation?: number;
        buffer?: number;
        amplitude?: number;
        speed?: number;
        period?: number;
        tilesX?: number;
        tilesY?: number;
        class?: string;
        style?: string;
    };

    let {
        svg,
        mode = 'none',
        fit = 'natural',
        complexity = 6,
        interpolation = 50,
        buffer = 20,
        amplitude = 24,
        speed = 1,
        period = 600,
        tilesX = 1,
        tilesY = 1,
        class: className = '',
        style = ''
    }: Props = $props();

    // Functions can't cross the Astro client:only boundary (props are JSON-serialized),
    // so the animator lives here and is selected by `mode`.
    function animateFor(
        mode: Mode,
        amplitude: number,
        speed: number,
        period: number,
    ): ((base: number[][], t: number) => number[][]) | null {
        if (mode === 'wave') {
            // single sine wave riding along x — the whole shape ripples horizontally
            return (base, t) =>
                base.map(([x, y]) => [x, y + amplitude * Math.sin(x / period + t * speed)]);
        }
        if (mode === 'jelly') {
            // each control point oscillates with its own phase — organic wobble
            return (base, t) =>
                base.map(([x, y], i) => {
                    const phase = i * 0.55;
                    return [
                        x + amplitude * Math.sin(t * speed + phase),
                        y + amplitude * 0.85 * Math.cos(t * speed * 1.3 + phase * 1.3),
                    ];
                });
        }
        return null;
    }

    let host: HTMLDivElement;

    // Mean-value coordinate warp helpers (port of PavelLaptev/Figma-Warp utils)
    function createPointsArray(width: number, height: number, amount: number, controlBuffer: number) {
        const dot = (length: number, i: number) => (length / amount) * i;
        const idx = [...Array(amount).keys()];

        const left = idx.map((_, i) => [0, dot(height, i)]);
        const bottom = idx.map((_, i) => [dot(width, i), height]);
        const right = idx.map((_, i) => [width, dot(height, i + 1)]).reverse();
        const top = idx.map((_, i) => [dot(width, i + 1), 0]).reverse();

        const points = [...left, ...bottom, ...right, ...top];
        for (const p of points) {
            if (p[0] === 0) p[0] -= controlBuffer;
            if (p[1] === 0) p[1] -= controlBuffer;
            if (p[0] === width) p[0] += controlBuffer;
            if (p[1] === height) p[1] += controlBuffer;
        }
        return points;
    }

    // First pass: compute mean-value weights for each SVG point relative to V
    function warpIt(warp: any, points: number[][]) {
        warp.transform(function (v0: number[], V: number[][] = points) {
            const A: number[] = [];
            const W: number[] = [];
            const L: number[] = [];

            for (let i = 0; i < V.length; i++) {
                const j = (i + 1) % V.length;
                const r0i = Math.hypot(v0[0] - V[i][0], v0[1] - V[i][1]);
                const r0j = Math.hypot(v0[0] - V[j][0], v0[1] - V[j][1]);
                const rij = Math.hypot(V[i][0] - V[j][0], V[i][1] - V[j][1]);
                const dn = 2 * r0i * r0j;
                const r = (r0i ** 2 + r0j ** 2 - rij ** 2) / dn;
                A[i] = isNaN(r) ? 0 : Math.acos(Math.max(-1, Math.min(r, 1)));
            }

            for (let j = 0; j < V.length; j++) {
                const i = (j > 0 ? j : V.length) - 1;
                const r = Math.hypot(V[j][0] - v0[0], V[j][1] - v0[1]);
                W[j] = (Math.tan(A[i] / 2) + Math.tan(A[j] / 2)) / r;
            }

            const Ws = W.reduce((a, b) => a + b, 0);
            for (let i = 0; i < V.length; i++) L[i] = W[i] / Ws;
            return [...v0, ...L];
        });
    }

    // Subsequent passes: re-derive (x, y) from the (possibly moved) polygon V
    function warpReposition(warp: any, points: number[][]) {
        warp.transform(([, , ...W]: number[], V: number[][] = points) => {
            let nx = 0, ny = 0;
            for (let i = 0; i < V.length; i++) {
                nx += W[i] * V[i][0];
                ny += W[i] * V[i][1];
            }
            return [nx, ny, ...W];
        });
    }

    onMount(() => {
        host.innerHTML = svg;
        const svgEl = host.querySelector('svg') as SVGSVGElement | null;
        if (!svgEl) return;

        // 'stretch' fills the container regardless of source ratio (good for the
        // background watermark). 'natural' preserves the source aspect ratio and
        // lets warped points spill outside the viewBox so the wave isn't clipped.
        if (fit === 'stretch') {
            svgEl.setAttribute('preserveAspectRatio', 'none');
        } else {
            svgEl.style.overflow = 'visible';
        }
        svgEl.style.display = 'block';
        svgEl.style.width = '100%';
        svgEl.style.height = '100%';
        svgEl.removeAttribute('width');
        svgEl.removeAttribute('height');

        const vb = svgEl.viewBox.baseVal;
        const tileW = vb && vb.width ? vb.width : 0;
        const tileH = vb && vb.height ? vb.height : 0;
        if (!tileW || !tileH) return;

        // Tile via cloned <g transform> blocks. Each cloned path goes through
        // the same warp transform — for tile borders to line up, the warp
        // must be position-periodic with period (tileW, tileH); use mode='tide'.
        if (tilesX > 1 || tilesY > 1) {
            const innerHTML = svgEl.innerHTML;
            const fragments: string[] = [];
            for (let y = 0; y < tilesY; y++) {
                for (let x = 0; x < tilesX; x++) {
                    if (x === 0 && y === 0) continue;
                    fragments.push(`<g transform="translate(${x * tileW},${y * tileH})">${innerHTML}</g>`);
                }
            }
            svgEl.insertAdjacentHTML('beforeend', fragments.join(''));
            svgEl.setAttribute('viewBox', `0 0 ${tileW * tilesX} ${tileH * tilesY}`);
        }

        const warp = new Warp(svgEl);
        warp.interpolate(interpolation);

        let raf = 0;
        const start = performance.now();

        if (mode === 'tide') {
            // Per-point sine displacement, periodic with (tileW, tileH) so the
            // warp at (x, y) equals the warp at (x + tileW, y + tileH) — tile
            // edges line up and the slide loop is seamless.
            const tau = Math.PI * 2;
            warp.transform(([x, y]: number[]) => [x, y, x, y]); // stash original

            // Cursor-driven phase nudge — adds a subtle reactive warp to the bg.
            let mouseX = 0, mouseY = 0;
            let smoothMx = 0, smoothMy = 0;
            const handleMove = (e: MouseEvent) => {
                mouseX = e.clientX / window.innerWidth - 0.5;  // -0.5..0.5
                mouseY = e.clientY / window.innerHeight - 0.5;
            };
            window.addEventListener('mousemove', handleMove, { passive: true });

            function frame(now: number) {
                const t = (now - start) / 1000;
                // Lerp toward the latest cursor pos so the response feels organic
                smoothMx += (mouseX - smoothMx) * 0.06;
                smoothMy += (mouseY - smoothMy) * 0.06;
                const mxPhase = smoothMx * 3.0; // ~½ cycle of phase nudge
                const myPhase = smoothMy * 3.0;
                const mPushX = smoothMx * amplitude * 1.4; // direct push toward cursor
                const mPushY = smoothMy * amplitude * 1.4;

                warp.transform((p: number[]) => {
                    const ox = p[2], oy = p[3];
                    const phaseX = (tau * ox) / tileW;
                    const phaseY = (tau * oy) / tileH;
                    const dx = amplitude * (Math.sin(phaseX + t * speed + mxPhase) + 0.5 * Math.cos(phaseY + t * speed * 1.3)) + mPushX;
                    const dy = amplitude * (Math.cos(phaseY + t * speed * 0.7 + myPhase) + 0.5 * Math.sin(phaseX + t * speed * 1.1)) + mPushY;
                    return [ox + dx, oy + dy, ox, oy];
                });
                raf = requestAnimationFrame(frame);
            }
            raf = requestAnimationFrame(frame);

            return () => {
                cancelAnimationFrame(raf);
                window.removeEventListener('mousemove', handleMove);
            };
        } else if (mode === 'wave' || mode === 'jelly') {
            const w = tileW * tilesX;
            const h = tileH * tilesY;
            const base = createPointsArray(w, h, complexity, buffer);
            let pts = base.map(p => [...p]);

            warpIt(warp, pts);
            warpReposition(warp, pts);

            const animate = animateFor(mode, amplitude, speed, period);
            if (!animate) return;

            function frame(now: number) {
                const t = (now - start) / 1000;
                pts = animate!(base, t);
                warpReposition(warp, pts);
                raf = requestAnimationFrame(frame);
            }
            raf = requestAnimationFrame(frame);
        } else {
            return;
        }

        return () => cancelAnimationFrame(raf);
    });
</script>

<div bind:this={host} class={className} {style}></div>
