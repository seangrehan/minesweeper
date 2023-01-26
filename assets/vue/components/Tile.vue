<template>
    <div class="flex items-center justify-center bg-slate-300 w-10 h-10" :class="getClass()">
        <div v-if="tile.isRevealed && tile.isBomb">
            💣
        </div>
        <div v-if="tile.hasFlag">
            ⛳
        </div>
        <div class="font-bold" v-if="!tile.isBomb && tile.isRevealed && tile.totalNeighouringBombs > 0">
            {{ tile.totalNeighouringBombs }}
        </div>
    </div>
</template>

<script>
export default {
    name: "Tile",
    props: {
        tile: {
            type: Object,
            required: true,
        },
    },
    methods: {
        getClass() {
            const tile = this.tile;

            if (tile.isRevealed) {
                return 'uncovered';        
            }

            return 'covered';
        }
    }
}
</script>