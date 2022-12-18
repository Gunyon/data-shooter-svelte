<script lang="ts">
  import Header from './components/Header.svelte';

  let tab: 'last' | 'specific' | 'period' = 'last';

  let dateStart = new Date();
  let dateEnd = new Date();

  let reportKey = '';
  let accountId = '';
  let lastWorkingDay = '';
  let markets = [];
  let selectedMarket;

  const headers = {
    headers: {
      Authorization: `Bearer ${localStorage.getItem('authToken') || ''}`
    }
  };

  function handleError(response) {
    if (!response.ok) {
      if (response.status === 401) {
        window.location.replace(
          'https://www.energymarketprice.com/home/login?redirectUrl=/data-shooter-tmp'
        );
      } else {
        throw response;
      }
    } else return response;
  }

  fetch('https://www.energymarketprice.com/Export/DataShooter/Report', headers)
    // fetch the report key
    .then(response => handleError(response))
    .then(response => response.text())
    .then(response => {
      let parsedJson = JSON.parse(response);
      accountId = parsedJson.account;
      reportKey = parsedJson.key;
      lastWorkingDay = parsedJson.lastWorkedDate;
    })
    .then(() => {
      fetch(
        `https://www.energymarketprice.com/Export/DataShooter/GroupName/${reportKey}`,
        headers
      )
        // fetch the individual report names
        .then(response => response.text())
        .then(response => {
          markets = JSON.parse(response);
        });
    })
    .catch(console.error);

  function downloadFile(type) {
    const extensions = {
      XML: 'xml',
      EXCEL: 'xlsx',
      JSON: 'json'
    };
    const url = [
      'https://www.energymarketprice.com/Export/DataShooter',
      type,
      reportKey,
      selectedMarket || '',
      dateStart || '',
      dateEnd || ''
    ]
      .filter(Boolean)
      .join('/');
    fetch(url, headers)
      .then(response => handleError(response))
      .then(response => response.blob())
      .then(file => {
        let filename = `data-shooter.${extensions[type]}`;
        const linkElement = document.createElement('a');
        linkElement.setAttribute('href', window.URL.createObjectURL(file));
        linkElement.setAttribute('download', filename);
        linkElement.click();
      })
      .catch(console.error);
  }
</script>

<Header />

<main>
  <div class="main-wrapper">
    <h1 class="title">Get Your Data Shooter Link and Markets</h1>

    <div class="tabs">
      <button
        class="tab-btn"
        class:active={tab === 'last'}
        on:click={() => (tab = 'last')}
      >
        Last Working Day
      </button>
      <button
        class="tab-btn"
        class:active={tab === 'specific'}
        on:click={() => (tab = 'specific')}
      >
        Specific Day
      </button>
      <button
        class="tab-btn"
        class:active={tab === 'period'}
        on:click={() => (tab = 'period')}
      >
        Specific period
      </button>
    </div>

    <section class="tab-content">
      {#if tab !== 'last'}
        <div class="calendar-container">
          <label class="label" for="date-start">From Date</label>
          <span class="calendar-icon">
            <input id="date-start" type="date" bind:value={dateStart} />
          </span>
        </div>
      {/if}

      {#if tab === 'period'}
        <div class="calendar-container">
          <label class="label" for="date-end">To Date</label>
          <span class="calendar-icon">
            <input id="date-end" type="date" bind:value={dateEnd} />
          </span>
        </div>
      {/if}

      <div>
        <label for="market" class="label">Select Group Name</label>
        <select id="market" class="select" bind:value={selectedMarket}>
          <option value="">All Markets</option>
          {#each markets as market}
            <option value={market.id}>{market.name}</option>
          {/each}
        </select>
      </div>

      <button class="emp-btn" on:click={() => downloadFile('XML')}>XML</button>
      <button class="emp-btn" on:click={() => downloadFile('JSON')}>JSON</button
      >
      <button class="emp-btn excel" on:click={() => downloadFile('EXCEL')}
        >EXCEL</button
      >
    </section>
  </div>
</main>
