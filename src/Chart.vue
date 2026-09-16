<template>
    <div class="bar">
        <canvas ref="charCanvas"></canvas>
    </div>
    <div class="pie">
        <canvas ref="charPie"></canvas>
    </div>

</template>

<script setup lang="ts">
import Chart from 'chart.js/auto';
import { ref, onMounted } from 'vue'
import { getRelativePosition } from 'chart.js/helpers';
import * as Utils from './utils.ts'

const DATA_COUNT = 5;
const NUMBER_CFG = { count: DATA_COUNT, min: 0, max: 100 };
const charCanvas = ref(null);
const charPie = ref(null);

const data = {
    labels: ['Red', 'Orange', 'Yellow', 'Green', 'Blue'],
    datasets: [
        {
            label: 'Dataset 1',
            data: Utils.numbers(NUMBER_CFG),
            backgroundColor: Object.values(Utils.CHART_COLORS),
            hoverOffset: 10,
        }
    ]
};
//let myChart = null;
onMounted(() => {
    //const ctx = document.getElementById('myChart') as HTMLCanvasElement;
    if (charCanvas.value) {
        new Chart(charCanvas.value, {
            type: 'bar',
            data: {
                labels: ['Silver', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
                datasets: [{
                    label: '# of Votes',
                    data: [133, 19, 3, 5, 2, 3],
                    borderWidth: 1
                },
                {
                    label: '# of Abstention',
                    data: [13, 59, 32, 1, 9, 30],
                    borderWidth: 1
                }
                ]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        });

    };
    if (charPie.value) {
        new Chart(charPie.value, {
            type: 'doughnut',
            data: data,
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'top',
                    },
                    title: {
                        display: true,
                        text: 'Chart.js Doughnut Chart'
                    }
                }
            },
        });
    }
})

</script>

<style>
.bar {
    position: relative;
    height: 50vh;
    width: 50vw;
}

.pie {
    position: relative;
    height: 50vh;
    width: 50vw;


}
</style>