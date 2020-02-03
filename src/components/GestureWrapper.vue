<script>
export default {
    props: {
        dirRatio: {
            type: Number,
            default: 0.5,
        },

        minDistance: {
            type: Number,
            default: 16,
        },

        /** Don't reset cssNumber on gesture end. */
        keepCssState: {
            type: Boolean,
        },
    },

    data() {
        return {
            isCreated: false,
            inGesture: false,
            isMove: false,
            wrapper: {
                startX: 0,
                startY: 0,
                moveX: 0,
                moveY: 0,
                endX: 0,
                endY: 0,
                offsetX: 0,
                offsetY: 0,
            },
            pageWidth: document.documentElement.clientWidth,
        };
    },

    computed: {
        slotProps() {
            return {
                wrapper: this.wrapper,
                cssNumber: this.cssNumber,
                inGesture: this.inGesture,
                isMove: this.isMove,
                isMoveLeft: this.isMoveLeft,
                isMoveRight: this.isMoveRight,
            };
        },

        handlers() {
            return {
                touchstart: this.onGestureStart,
                touchmove: this.onGestureMove,
                touchend: this.onGestureEnd,
                mousedown: this.onGestureStart,
                mousemove: this.onGestureMove,
                mouseup: this.onGestureEnd,
            };
        },

        isMoveX() {
            const offsetX = Math.abs(this.wrapper.moveX - this.wrapper.startX);
            const offsetY = Math.abs(this.wrapper.moveY - this.wrapper.startY);
            return offsetY < this.dirRatio * offsetX;
        },

        isMoveLeft() {
            return this.isMoveX && this.wrapper.startX > this.wrapper.moveX;
        },

        isMoveRight() {
            return this.isMoveX && this.wrapper.startX < this.wrapper.moveX;
        },

        /**
         * @returns {number} (unit interval) from 0 to 1
         */
        cssNumber() {
            if (!this.isMove && !this.keepCssState) return 0;
            const diff = Math.abs(this.wrapper.startX - this.wrapper.moveX);
            return diff / this.pageWidth;
        },
    },

    created() {
        this.toggleHandlers();
    },

    beforeDestroy() {
        this.toggleHandlers();
    },

    methods: {
        onGestureStart(evt) {
            this.inGesture = true;

            const resolvedEvt = this.resolveEvent(evt);

            this.wrapper.startX = Math.ceil(resolvedEvt.clientX);
            this.wrapper.startY = Math.ceil(resolvedEvt.clientY);

            this.$emit('start', this.wrapper);
        },

        onGestureMove(evt) {
            if (!this.inGesture) return;

            this.isMove = true;

            const resolvedEvt = this.resolveEvent(evt);

            this.wrapper.moveX = Math.ceil(resolvedEvt.clientX);
            this.wrapper.moveY = Math.ceil(resolvedEvt.clientY);

            this.$emit('move', this.wrapper);
        },

        onGestureEnd(evt) {
            this.inGesture = false;
            this.isMove = false;

            const resolvedEvt = this.resolveEvent(evt);

            this.wrapper.endX = Math.ceil(resolvedEvt.clientX);
            this.wrapper.endY = Math.ceil(resolvedEvt.clientY);

            this.$emit('end', this.wrapper);
            this.resolveGesture();
        },

        resolveEvent(evt) {
            return /^touch/.test(evt.type) ? evt.changedTouches[0] : evt;
        },

        resolveGesture() {
            const wrapper = this.wrapper;
            const { startX, startY, endX, endY } = wrapper;

            wrapper.offsetX = Math.abs(endX - startX);
            wrapper.offsetY = Math.abs(endY - startY);

            if (wrapper.offsetY < this.dirRatio * wrapper.offsetX) {
                endX < startX - this.minDistance && this.$emit('left', wrapper);
                endX > startX + this.minDistance &&
                    this.$emit('right', wrapper);
            }

            if (wrapper.offsetX < this.dirRatio * wrapper.offsetY) {
                endY < startY - this.minDistance && this.$emit('up', wrapper);
                endY > startY + this.minDistance && this.$emit('down', wrapper);
            }
        },

        toggleHandlers() {
            this.isCreated = !this.isCreated;
            const action = this.isCreated ? 'add' : 'remove';

            Object.keys(this.handlers).forEach((eventName) => {
                document[`${action}EventListener`](
                    eventName,
                    this.handlers[eventName],
                    false,
                );
            });
        },
    },

    render() {
        return this.$scopedSlots.default({ ...this.slotProps });
    },
};
</script>
