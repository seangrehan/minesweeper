<template>
    <section>
        {{ timeElapsed.toFixed(1) }}
    </section>
</template>

<script>
export default {
    name: 'timer',
    methods: {
        startTimer() {
            if (this.gameFinished) {
                window.cancelAnimationFrame(this.timerReq);
                return; 
            }

            this.timer = false;
            this.timerReq = window.requestAnimationFrame(this.setTimer);
        },
        setTimer(timestamp) {
            if (this.timer === false) {
                this.timer = timestamp;
            }

            this.timeElapsed = (timestamp - this.timer) / 1000;

            if (this.gameFinished === false) {
                window.requestAnimationFrame(this.setTimer);
            }
        }
    },
    props: {
        gameFinished: {
            type: Boolean,
            required: true,
        },
    },
    watch: {
        finished() {
            this.startTimer();
        },
    },
    data() {
        return {
            timeElapsed: 0,
            timer: false,
            timerReq: false,
        }
    },
    mounted() {
        this.startTimer();
    },

}
</script>