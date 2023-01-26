<script>
import Tile from './Tile.vue';
import Timer from './Timer.vue';

export default {
    name: "Board",
    components: {
        Tile,
        Timer,
    },
    props: {
        bombs: {
            type: Number,
            default: 10,
        },
        columns: {
            type: Number,
            default: 9,
        },
        rows: {
            type: Number,
            default: 9,
        },
    },
    methods: {
        setupBoard() {
            const board = [];
            const totalTiles = this.rows * this.columns;

            for (let tile = 0; tile < totalTiles; tile++) {
                board.push({
                    isBomb: false,
                    isRevealed: false,
                    hasFlag: false,
                    neighbours: [],
                    totalBombs: 0,
                    totalNeighouringBombs: 0,
                });
            }

            // Set bombs
            let { bombs } = this;
            let bombsIndex = [];

            while (bombs > 0) {
                const index = Math.floor(Math.random() * totalTiles);

                if (board[index].isBomb === false) {
                    board[index].isBomb = true;
                    bombsIndex.push(index);
                    bombs--;
                }
            }

            // Set total neighbouring bombs
            bombsIndex.forEach(tileIndex => {
                const isNotTopRow = tileIndex > (this.columns - 1);
                const isNotBottomRow = tileIndex < ((totalTiles - 1) - this.columns);
                const isOnFarLeft = (tileIndex % this.columns) === 0;
                const isOnFarRight = (tileIndex % (this.columns - 1)) === 0;

                if (isNotTopRow) {
                    if (!isOnFarLeft) {
                        board[tileIndex - (this.columns + 1)].totalNeighouringBombs++;
                    }

                    board[tileIndex - this.columns].totalNeighouringBombs++;

                    if (!isOnFarRight) {
                        board[tileIndex - (this.columns - 1)].totalNeighouringBombs++;
                    }
                }

                if (!isOnFarLeft) {
                    board[tileIndex - 1].totalNeighouringBombs++;
                }

                if (!isOnFarRight) {
                    board[tileIndex + 1].totalNeighouringBombs++;
                }

                if (isNotBottomRow) {
                    if (!isOnFarLeft) {
                        board[tileIndex + (this.columns - 1)].totalNeighouringBombs++;
                    }

                    board[tileIndex + this.columns].totalNeighouringBombs++;

                    if (!isOnFarRight) {
                        board[tileIndex + (this.columns + 1)].totalNeighouringBombs++;
                    }
                }

            });

            this.board = board;

            this.gameFinished = true;

            this.$nextTick(() => {
                this.gameFinished = false;
            });

            this.gameWon = false;
            this.remainingBombs = this.bombs;
        },
        checkIfGameWon() {
            if (this.remainingBombs > 0 || this.gameFinished) {
                return;
            }

            const remainingBoardTiles = this.board.find(
                tile => tile.isRevealed === false && tile.hasFlag === false
            );

            if (remainingBoardTiles.length === 0) {
                this.gameFinished = true;
                this.gameWon = true;
            }
        },
        addFlag(tile) {
            if (this.gameFinished || tile.isRevealed) {
                return;
            }

            tile.hasFlag = !tile.hasFlag;

            let board = this.board;

            const flagCount = board.reduce((accumulator, currentTile) => {
                if (currentTile.hasFlag) {
                    return accumulator++;
                }

                return accumulator;
            }, 0);

            this.remainingBombs = this.bombs - flagCount;

            this.checkIfGameWon();
        },
        doubleClickTile(tile, index) {
            if (this.gameFinished || tile.isRevealed === false) {
                return;
            }

            this.setAdjacentTiles(tile, index);

            if (tile.totalBombs < 1) {
                return;
            }

            let board = this.board;

            let totalFlags = 0;

            tile.neighbours.forEach(
                tileIndex => {
                    if (board[tileIndex].hasFlag) {
                        totalFlags++;
                    }
                }
            );

            if (totalFlags === tile.totalBombs) {
                this.clearAdjacentTiles(tile, true);
            }
        },
        clickTile(tile, index) {
            if (this.gameFinished || tile.hasFlag || tile.isRevealed) {
                return;
            }

            if (tile.isBomb) {
                let board = this.board;

                board.forEach(currentTile => {
                    if (currentTile.isBomb) {
                        currentTile.isRevealed = true;
                    }
                });

                this.gameFinished = true;

                return;
            }

            this.setAdjacentTiles(tile, index);

            tile.isRevealed = true;

            this.clearAdjacentTiles(tile);
            this.checkIfGameWon();
        },
        clearAdjacentTiles(tile, clear = false) {
            if (tile.totalNeighouringBombs > 0 && clear === false) {
                return;
            }

            const { board } = this;

            tile.neighbours.forEach(index => {
                if (board[index].isBomb === false) {
                    this.clickTile(board[index], index);
                }
            });
        },
        setAdjacentTiles(tile, index) {
            if (tile.neighbours.length > 0) {
                return;
            }

            const { board } = this;

            let neighbours = [];

            let totalBombs = 0;

            for (let x = -1; x < 2; x++) {
                for (let y = -1; y < 2; y++) {
                    const tileIndex = this.getTileIndex(index, x, y);

                    if (tileIndex) {
                        neighbours.push(tileIndex);

                        if (board[tileIndex].isBomb) {
                            totalBombs++;
                        }
                    }
                }
            }

            tile.totalBombs = totalBombs;
            tile.neighbours = neighbours;
        },
        getTileIndex(index, x, y) {
            const columns = this.columns;
            const rows = this.rows;

            if (x === 0 && y === 0) {
                return false;
            }

            if ((index % columns) + x < 0 || (index % columns) + x >= columns) {
                return false;
            }

            if (Math.floor(index / columns) + y < 0 || Math.floor(index / columns) + y >= rows) {
                return false;
            }

            return index + (y * columns + x);
        }
    },
    data() {
        return {
            board: [],
            gameFinished: false,
            gameWon: false,
            remainingBombs: 0,
        }
    },
    mounted() {
        this.setupBoard();
    },
}
</script>

<template>
    <section class="block w-full">
        {{ this.remainingBombs }} Bombs
        <Timer :gameFinished="gameFinished" />
    </section>

    <section class="grid" :style="{ 'grid-template-columns': 'repeat(' + columns + ', auto)' }">
        <Tile
            v-for="(tile, index) in board"
            :key="index"
            :tile="tile"
            @click.native="clickTile(tile, index)"
            @click.right.native="addFlag(tile)"
            @dblclick.native.prevent="doubleClickTile(tile, index)"
            @contextmenu.native.prevent
        />
    </section>
</template>
