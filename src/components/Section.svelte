<script lang="ts">
    const { title, children } = $props();
</script>

<div class="section">
    <h3 class="header">{title}</h3>
    <div class="inner">
        {@render children()}
    </div>
</div>

<style>
    .section {
        position: relative;
        width: 100%;
        min-height: 240px;

        overflow: auto;
    }

    .section .header {
        /* ABOUT */

        position: absolute;
        left: 8px;
        top: 8px;

        font-family: 'Rubik';
        font-style: normal;
        font-weight: 700;
        font-size: 48px;
        line-height: 100%;

        margin: 0;

        color: rgba(255, 238, 204, 0.4);

        transform: rotate(-90deg);
        transform-origin: top right;
        translate: -100% 25%;
    }

    .section .inner {
        position: relative;
        width: 70%;

        left: 80px;
        top: 32px;

        font-family: 'Bricolage Grotesque';
        font-style: normal;
        font-weight: 400;
        font-size: 20px;
        line-height: 100%;

        color: #FFEECC;
    }
</style>