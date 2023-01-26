<script>
import Board from './../components/Board.vue';

export default {
    components: {
        Board
    }
}
</script>

<template>
    <header class="flex w-full py-10 px-6">
        <h1 class="w-full text-5xl">Minesweeper</h1>
    </header>
    <main class="grid w-full justify-center">
        <Board />
    </main>
    <footer class="flex w-full">

    </footer>
</template>
