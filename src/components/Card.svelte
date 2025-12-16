<script lang="ts">
    const { logo, style, href } = $props();
</script>

<a class="card" href={href}>
    <div class="inner" style={style}>
        <img class="logo" src={logo} alt="Card Logo" />
    </div>
</a>

<style>
    .card {
        border: none;
        cursor: pointer;

        transform: translateY(8px) translateX(-8px);

        background: rgba(0, 0, 0, 0.25);
        
        border-radius: 16px;
    }

    .inner {
        border: 4px solid #47425E;
        border-radius: 16px;

        transform: translateY(-4px) translateX(-4px);
        transition: transform 600ms cubic-bezier(0.3, 0.7, 0.4, 1);        

        padding: 32px;
    }

    .card:hover .inner {
        transition: transform 600ms cubic-bezier(0.3, 0.7, 0.4, 1);        
        transform: translateY(-0px) translateX(0px);

        cursor: pointer;
    }
</style>