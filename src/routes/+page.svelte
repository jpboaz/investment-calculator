<!-- src/routes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { Line, Pie } from 'svelte-chartjs';
  import { 
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    ArcElement,
    Title,
    Tooltip,
    Legend
  } from 'chart.js';

  ChartJS.register(
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    ArcElement,
    Title,
    Tooltip,
    Legend
  );

  let principal = 1000;
  let interestRate = 5;
  let years = 10;
  let compoundingFrequency = 12;
  let monthlyContribution = 100;
  let darkMode = false;

  let lineChartData: any;
  let lineChartOptions: any;
  let pieChartData: any;
  let pieChartOptions: any;

  $: {
    const growthData = calculateInvestmentGrowth(principal, interestRate, years, compoundingFrequency, monthlyContribution);
    
    lineChartData = {
      labels: growthData.map((_, index) => index),
      datasets: [
        {
          label: 'Investment Value',
          data: growthData,
          borderColor: darkMode ? 'rgb(134, 239, 172)' : 'rgb(75, 192, 192)',
          backgroundColor: darkMode ? 'rgba(134, 239, 172, 0.1)' : 'rgba(75, 192, 192, 0.1)',
          tension: 0.1
        }
      ]
    };

    lineChartOptions = {
      responsive: true,
      plugins: {
        legend: {
          position: 'top' as const,
          labels: {
            color: darkMode ? '#E5E7EB' : '#111827'
          }
        },
        title: {
          display: true,
          text: 'Investment Growth Over Time',
          color: darkMode ? '#E5E7EB' : '#111827'
        }
      },
      scales: {
        x: {
          title: {
            display: true,
            text: 'Months',
            color: darkMode ? '#E5E7EB' : '#111827'
          },
          grid: {
            color: darkMode ? 'rgba(255, 255, 255, 0.1)' : 'rgba(0, 0, 0, 0.1)'
          },
          ticks: {
            color: darkMode ? '#E5E7EB' : '#111827'
          }
        },
        y: {
          title: {
            display: true,
            text: 'Value ($)',
            color: darkMode ? '#E5E7EB' : '#111827'
          },
          grid: {
            color: darkMode ? 'rgba(255, 255, 255, 0.1)' : 'rgba(0, 0, 0, 0.1)'
          },
          ticks: {
            color: darkMode ? '#E5E7EB' : '#111827'
          }
        }
      }
    };

    const totalContributions = principal + (monthlyContribution * years * 12);
    const totalInterest = growthData[growthData.length - 1] - totalContributions;

    pieChartData = {
      labels: ['Total Contributions', 'Total Interest'],
      datasets: [
        {
          data: [totalContributions, totalInterest],
          backgroundColor: darkMode 
            ? ['rgba(134, 239, 172, 0.8)', 'rgba(59, 130, 246, 0.8)']
            : ['rgba(75, 192, 192, 0.8)', 'rgba(153, 102, 255, 0.8)'],
          borderColor: darkMode ? '#374151' : '#FFFFFF',
          borderWidth: 1
        }
      ]
    };

    pieChartOptions = {
      responsive: true,
      plugins: {
        legend: {
          position: 'top' as const,
          labels: {
            color: darkMode ? '#E5E7EB' : '#111827'
          }
        },
        title: {
          display: true,
          text: 'Contributions vs Interest',
          color: darkMode ? '#E5E7EB' : '#111827'
        }
      }
    };
  }

  function calculateInvestmentGrowth(principal: number, rate: number, years: number, compoundingFrequency: number, monthlyContribution: number): number[] {
    const monthlyRate = (rate / 100) / 12;
    const totalMonths = years * 12;
    const growthData = [principal];

    for (let month = 1; month <= totalMonths; month++) {
      const previousValue = growthData[month - 1];
      const interestEarned = previousValue * monthlyRate;
      const newValue = previousValue + interestEarned + monthlyContribution;
      growthData.push(newValue);
    }

    return growthData;
  }

  $: futureValue = calculateInvestmentGrowth(principal, interestRate, years, compoundingFrequency, monthlyContribution).pop() || 0;

  function formatCurrency(value: number): string {
    return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value);
  }

  function toggleDarkMode() {
    darkMode = !darkMode;
    document.documentElement.classList.toggle('dark');
  }

  onMount(() => {
    if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
      darkMode = true;
      document.documentElement.classList.add('dark');
    }
  });
</script>

<div class={`min-h-screen transition-colors duration-300 ${darkMode ? 'dark:bg-gray-900' : 'bg-white'}`}>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-4">
      <h1 class="text-3xl font-bold dark:text-white">Investment Calculator</h1>
      <button 
        on:click={toggleDarkMode}
        class="px-4 py-2 rounded-md bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white"
      >
        {darkMode ? '☀️ Light' : '🌙 Dark'}
      </button>
    </div>
    
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <div>
        <label class="block mb-2 dark:text-white" for="principal">Initial Investment</label>
        <input type="number" id="principal" bind:value={principal} min="0" step="100" class="w-full p-2 border rounded dark:bg-gray-700 dark:text-white dark:border-gray-600">
      </div>
      
      <div>
        <label class="block mb-2 dark:text-white" for="interestRate">Annual Interest Rate (%)</label>
        <input type="number" id="interestRate" bind:value={interestRate} min="0" step="0.1" class="w-full p-2 border rounded dark:bg-gray-700 dark:text-white dark:border-gray-600">
      </div>
      
      <div>
        <label class="block mb-2 dark:text-white" for="years">Investment Period (years)</label>
        <input type="number" id="years" bind:value={years} min="1" step="1" class="w-full p-2 border rounded dark:bg-gray-700 dark:text-white dark:border-gray-600">
      </div>
      
      <div>
        <label class="block mb-2 dark:text-white" for="compoundingFrequency">Compounding Frequency (per year)</label>
        <select id="compoundingFrequency" bind:value={compoundingFrequency} class="w-full p-2 border rounded dark:bg-gray-700 dark:text-white dark:border-gray-600">
          <option value="1">Annually</option>
          <option value="2">Semi-annually</option>
          <option value="4">Quarterly</option>
          <option value="12">Monthly</option>
          <option value="365">Daily</option>
        </select>
      </div>

      <div>
        <label class="block mb-2 dark:text-white" for="monthlyContribution">Monthly Contribution</label>
        <input type="number" id="monthlyContribution" bind:value={monthlyContribution} min="0" step="10" class="w-full p-2 border rounded dark:bg-gray-700 dark:text-white dark:border-gray-600">
      </div>
    </div>
    
    <div class="mt-8 p-4 bg-green-100 dark:bg-green-900 rounded">
      <h2 class="text-2xl font-bold mb-2 dark:text-white">Future Value</h2>
      <p class="text-3xl font-bold text-green-700 dark:text-green-300">{formatCurrency(futureValue)}</p>
    </div>

    <div class="mt-8">
      <Line data={lineChartData} options={lineChartOptions} />
    </div>

    <div class="mt-8">
      <Pie data={pieChartData} options={pieChartOptions} />
    </div>
  </div>
</div>
