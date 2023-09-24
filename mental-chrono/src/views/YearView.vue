<template>
    <svg :width="width" :height="height" :viewBox="currentViewBox" @wheel="onWheel" @mousedown="startDrag" @mousemove="drag"
        @mouseup="endDrag" @mouseleave="endDrag" xmlns="http://www.w3.org/2000/svg">
        <g v-for="i in 52" :key="i" :transform="`rotate(${i * segmentAngle} 400 450)`">
            <path :d="getSegmentPath()" class="week-placeholder" />
            <WeekView v-if="zoomLevel >= 0 && visibleWeeks.includes(i)"></WeekView>
        </g>
    </svg>
</template>
  
<script>
import WeekView from './WeekView.vue';

export default {
    components: {
        WeekView
    },

    data() {
        return {
            width: '95vw',
            height: '90vh',
            segmentAngle: 360 / 52,
            outerRadiusX: 400,
            innerRadiusX: 370,
            outerRadiusY: 450,
            innerRadiusY: 420,
            zoomLevel: 1,
            viewBoxWidth: 800,
            viewBoxHeight: 900,
            currentViewBox: '0 0 800 900',
            dragging: false,
            lastX: 0,
            lastY: 0,
        }
    },
    computed: {
        visibleWeeks() {
            const weeks = [];
            for (let i = 0; i < 52; i++) {
                const angle = i * this.segmentAngle;

                // Calculate the bounding box for the segment
                const minX = 400 + this.innerRadiusX * Math.cos(Math.PI * angle / 180);
                const minY = 450 + this.innerRadiusY * Math.sin(Math.PI * angle / 180);
                const maxX = 400 + this.outerRadiusX * Math.cos(Math.PI * angle / 180);
                const maxY = 450 + this.outerRadiusY * Math.sin(Math.PI * angle / 180);

                const viewBox = this.currentViewBox.split(" ").map(Number);

                // Check if the bounding box intersects with the viewBox
                if (maxX >= viewBox[0] && minX <= viewBox[0] + viewBox[2] && maxY >= viewBox[1] && minY <= viewBox[1] + viewBox[3]) {
                    weeks.push(i);
                }
            }
            console.log(weeks);
            return weeks;
        }
    },
    methods: {
        getSegmentPath() {
            const startAngle = 0;
            const endAngle = this.segmentAngle;
            const largeArcFlag = endAngle - startAngle <= 180 ? 0 : 1;

            const x1 = 400 + this.innerRadiusX * Math.cos(Math.PI * startAngle / 180);
            const y1 = 450 + this.innerRadiusY * Math.sin(Math.PI * startAngle / 180);

            const x2 = 400 + this.outerRadiusX * Math.cos(Math.PI * startAngle / 180);
            const y2 = 450 + this.outerRadiusY * Math.sin(Math.PI * startAngle / 180);

            const x3 = 400 + this.outerRadiusX * Math.cos(Math.PI * endAngle / 180);
            const y3 = 450 + this.outerRadiusY * Math.sin(Math.PI * endAngle / 180);

            const x4 = 400 + this.innerRadiusX * Math.cos(Math.PI * endAngle / 180);
            const y4 = 450 + this.innerRadiusY * Math.sin(Math.PI * endAngle / 180);

            return `
          M ${x1} ${y1}
          L ${x2} ${y2}
          A ${this.outerRadiusX} ${this.outerRadiusY} 0 ${largeArcFlag} 1 ${x3} ${y3}
          L ${x4} ${y4}
          A ${this.innerRadiusX} ${this.innerRadiusY} 0 ${largeArcFlag} 0 ${x1} ${y1}
        `;
        },
        onWheel(event) {
            event.preventDefault();

            // Adjust the zoom level based on the wheel delta
            this.zoomLevel += event.deltaY * -0.01;

            // Clamp the zoom level to reasonable values
            this.zoomLevel = Math.min(Math.max(0.8, this.zoomLevel), 20);

            // Calculate new viewBox dimensions based on zoom level
            const newWidth = this.viewBoxWidth / this.zoomLevel;
            const newHeight = this.viewBoxHeight / this.zoomLevel;

            // Adjust viewBox to center on mouse position
            const mouseXPercentage = event.offsetX / parseInt(this.width);
            const mouseYPercentage = event.offsetY / parseInt(this.height);

            const newX = (this.viewBoxWidth - newWidth) * mouseXPercentage * 0.1;
            const newY = (this.viewBoxHeight - newHeight) * mouseYPercentage * 0.1;

            this.currentViewBox = `${newX} ${newY} ${newWidth} ${newHeight}`;
            console.log(this.zoomLevel);
        },
        startDrag(event) {
            this.dragging = true;
            this.lastX = event.clientX;
            this.lastY = event.clientY;
        },
        drag(event) {
            if (!this.dragging) return;

            const dx = event.clientX - this.lastX;
            const dy = event.clientY - this.lastY;

            const viewBox = this.currentViewBox.split(" ").map(Number);

            const scaleX = this.viewBoxWidth / parseInt(this.width);
            const scaleY = this.viewBoxHeight / parseInt(this.height);

            viewBox[0] -= dx * scaleX * 0.1;
            viewBox[1] -= dy * scaleY * 0.1;

            this.currentViewBox = viewBox.join(" ");

            this.lastX = event.clientX;
            this.lastY = event.clientY;
        },
        endDrag() {
            this.dragging = false;
        }
    }
}
</script>
  
<style scoped>
.week-placeholder {
    fill: gray;
}

.week-placeholder:hover {
    fill: red;
}
</style>
  