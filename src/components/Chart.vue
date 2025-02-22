<template>
  <div class="relative w-fit">
    <canvas id="chart"></canvas>
    <div
      class="
        absolute
        top-1/2
        left-1/2
        transform
        -translate-x-1/2 -translate-y-1/2
      "
    >
      <p class="text-center font-semibold text-lg text-neutral-600">
        {{ asCurrency(grossIncome[displayFrequency]) }}
      </p>
      <small class="text-small">gross income</small>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, watch } from "vue";
import { Chart, registerables } from "chart.js";

import { storeToRefs } from "pinia";
import { asCurrency, asPercentage } from "@/utils.js";

import { useTaxesStore } from "@/stores";
import ChartDataLabels from "chartjs-plugin-datalabels";

const { grossIncome, netIncome, irsPay, ssPay, colors, displayFrequency } =
  storeToRefs(useTaxesStore());

Chart.register(...registerables);
Chart.register(ChartDataLabels);

let chart;
onMounted(() => {
  buildChart();
});

const chartData = computed(() => {
  return {
    labels: ["Net Income", "IRS", "SS"],
    datasets: [
      {
        data: [
          netIncome.value[displayFrequency.value],
          irsPay.value[displayFrequency.value],
          ssPay.value[displayFrequency.value],
        ],
        backgroundColor: [
          colors.value.netIncome,
          colors.value.irs,
          colors.value.ss,
        ],
        hoverOffset: 4,
      },
    ],
  };
});
const chartOptions = computed(() => {
  return {
    plugins: {
      legend: {
        display: false,
      },
      datalabels: {
        display: true,
        textAlign: "center",
        formatter: (val, ctx) => {
          return (
            ctx.chart.data.labels[ctx.dataIndex] +
            "\n" +
            asPercentage(val / grossIncome.value[displayFrequency.value])
          );
        },
        color: "#fff",
      },
      tooltip: {
        callbacks: {
          label: function (item) {
            return asCurrency(item.raw, 2);
          },
        },
      },
    },
    responsive: true,
  };
});

watch(
  () => chartData.value,
  (newData) => {
    newData.datasets.forEach((d, i) => {
      chart.data.datasets[i].data = d.data;
    });
    chart.update();
  }
);
const buildChart = () => {
  const canvas = document.getElementById("chart") as HTMLCanvasElement;
  const ctx = canvas.getContext("2d");
  chart = new Chart(ctx, {
    type: "doughnut",
    data: chartData.value,
    options: chartOptions.value,
  });
  chart.update();
};
</script>
