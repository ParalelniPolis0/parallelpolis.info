<script>
    let { id } = $props();
</script>

<iframe
    id="ytplayer"
    type="text/html"
    width="100%"
    height="100%"
    class="aspect-video bg-gray-200 dark:bg-gray-800 rounded"
    src="https://www.youtube.com/embed/{id}?autoplay=0&enablejsapi=1&playsinline=1&fs=1&origin=https://parallelpolis.info"
    frameborder="0"
></iframe>
