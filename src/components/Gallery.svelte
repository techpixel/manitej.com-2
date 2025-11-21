<div class="gallery">
    <svg width="1068" height="93" viewBox="0 0 1068 93" fill="none" xmlns="http://www.w3.org/2000/svg" class="lines">
        <line x1="2.9274" y1="89.7257" x2="83.9274" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="213.927" y1="89.7257" x2="294.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="563.927" y1="89.7257" x2="644.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="842.927" y1="89.7257" x2="923.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="73.9274" y1="89.7257" x2="154.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="423.927" y1="89.7257" x2="504.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="283.927" y1="89.7257" x2="364.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="631.927" y1="89.7257" x2="712.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="913.927" y1="89.7257" x2="994.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="143.927" y1="89.7257" x2="224.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="493.927" y1="89.7257" x2="574.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="773.927" y1="89.7257" x2="854.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="352.927" y1="89.7257" x2="433.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="703.927" y1="89.7257" x2="784.927" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
        <line x1="983.927" y1="89.7257" x2="1064.93" y2="2.72574" stroke="#FFEECC" stroke-width="8"/>
    </svg>
</div>

<style>
    .gallery {
        min-width: 40%;
        height: 266px;
        min-height: 266px;

        background-color: #02B9BE;

        overflow: hidden;

        height: 90px;

        z-index: 0;

        width: 45%;
    }

    .gallery .lines {
        width: 100%;
        height: 100%;

        transform: translateX(0px);
        scale: 6;

        animation: linesX 120s linear infinite;

        will-change: transform;
    }

    @keyframes linesX {
        from {
            transform: translateX(0px);
        }
        to {
            transform: translateX(254.4px);
        }
    }
</style>
