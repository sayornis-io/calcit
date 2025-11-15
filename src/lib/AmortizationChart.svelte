<script lang="ts">
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  interface AmortizationRow {
    month: number;
    payment: number;
    principal: number;
    interest: number;
    balance: number;
  }

  let { amortizationSchedule = [] }: { amortizationSchedule: AmortizationRow[] } =
    $props();

  let canvas: HTMLCanvasElement;
  let chart: Chart;

  const initChart = () => {
    if (!canvas || amortizationSchedule.length === 0) return;

    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    // Destroy existing chart if it exists
    if (chart) {
      chart.destroy();
    }

    // Prepare data for the chart
    const months = amortizationSchedule.map((row) => row.month);
    const principalData = amortizationSchedule.map((row) => row.principal);
    const interestData = amortizationSchedule.map((row) => row.interest);
    const balanceData = amortizationSchedule.map((row) => row.balance);

    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: months,
        datasets: [
          {
            label: "Principal",
            data: principalData,
            borderColor: "#3b82f6",
            backgroundColor: "rgba(59, 130, 246, 0.1)",
            borderWidth: 2,
            tension: 0,
            yAxisID: "y",
            pointRadius: 0,
            pointHoverRadius: 4,
          },
          {
            label: "Interest",
            data: interestData,
            borderColor: "#ef4444",
            backgroundColor: "rgba(239, 68, 68, 0.1)",
            borderWidth: 2,
            tension: 0,
            yAxisID: "y",
            pointRadius: 0,
            pointHoverRadius: 4,
          },
          {
            label: "Loan Balance",
            data: balanceData,
            borderColor: "#10b981",
            backgroundColor: "rgba(16, 185, 129, 0.1)",
            borderWidth: 2,
            tension: 0,
            yAxisID: "y1",
            pointRadius: 0,
            pointHoverRadius: 4,
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: true,
        interaction: {
          mode: "index",
          intersect: false,
        },
        plugins: {
          legend: {
            display: true,
            position: "top",
            labels: {
              boxWidth: 12,
              padding: 15,
              font: {
                size: 13,
              },
              usePointStyle: true,
            },
          },
          tooltip: {
            backgroundColor: "rgba(0, 0, 0, 0.8)",
            padding: 12,
            titleFont: {
              size: 13,
              weight: "bold",
            },
            bodyFont: {
              size: 12,
            },
            callbacks: {
              label: function (context) {
                let label = context.dataset.label || "";
                if (label) {
                  label += ": ";
                }
                if (context.parsed.y !== null) {
                  label += "$" + context.parsed.y.toFixed(2);
                }
                return label;
              },
            },
          },
        },
        scales: {
          x: {
            display: true,
            title: {
              display: true,
              text: "Month",
              font: {
                size: 13,
                weight: "bold",
              },
            },
            ticks: {
              font: {
                size: 11,
              },
            },
          },
          y: {
            type: "linear",
            display: true,
            position: "left",
            title: {
              display: true,
              text: "Principal & Interest ($)",
              font: {
                size: 13,
                weight: "bold",
              },
            },
            ticks: {
              font: {
                size: 11,
              },
              callback: function (value) {
                return "$" + (value as number).toLocaleString("en-US", {
                  maximumFractionDigits: 0,
                });
              },
            },
            grid: {
                drawOnChartArea: true,
            },
          },
          y1: {
            type: "linear",
            display: true,
            position: "right",
            title: {
              display: true,
              text: "Balance ($)",
              font: {
                size: 13,
                weight: "bold",
              },
            },
            ticks: {
              font: {
                size: 11,
              },
              callback: function (value) {
                return "$" + (value as number).toLocaleString("en-US", {
                  maximumFractionDigits: 0,
                });
              },
            },
            grid: {
              drawOnChartArea: false,
            },
          },
        },
      },
    });
  };

  onMount(() => {
    initChart();
    return () => {
      if (chart) {
        chart.destroy();
      }
    };
  });

  $effect(() => {
    if (amortizationSchedule.length > 0) {
      initChart();
    }
  });
</script>

<div class="w-full">
  <canvas bind:this={canvas}></canvas>
</div>
