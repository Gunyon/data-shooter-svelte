<script lang="ts">
  import logo from './assets/logo.svg';
  import logout from './assets/log-out.svg';

  let activeTab: 'last' | 'specific' | 'period' = 'last';

  const authToken = localStorage.getItem('authToken');
  const userName = localStorage.getItem('userName');

  let dateStart = new Date();
  let dateEnd = new Date();

  let reportKey = '';
  let accountId = '';
  let lastWorkingDay = '';
  let markets = [];
  let selectedMarket;

  const headers = {
    headers: {
      Authorization: `Bearer ${authToken}`
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
      .then(document => {
        let filename = `data-shooter.${extensions[type]}`;
        const linkElement = document.createElement('a');
        linkElement.setAttribute('href', window.URL.createObjectURL(document));
        linkElement.setAttribute('download', filename);
        linkElement.click();
      })
      .catch(console.error);
  }
</script>

<nav>
  <div class="nav-wrapper">
    <img class="logo" src={logo} alt="logo" />
    <div class="details">
      {#if userName}
        <p class="user-name-field">{userName}</p>
      {/if}
      <a
        class="log-out-link"
        href="https://www.energymarketprice.com/home/logout"
        ><img src={logout} class="log-out-icon" alt="logout icon" /> Logout</a
      >
    </div>
  </div>
</nav>

<main>
  <div class="main-wrapper">
    <h1 class="title">Get Your Data Shooter Link and Markets</h1>

    <div class="container">
      <ul>
        <li class:active={activeTab === 'last'}>
          <button on:click={() => (activeTab = 'last')}>
            Last Working Day
          </button>
        </li>
        <li class:active={activeTab === 'specific'}>
          <button on:click={() => (activeTab = 'specific')}>
            Specific Day
          </button>
        </li>
        <li class:active={activeTab === 'period'}>
          <button on:click={() => (activeTab = 'period')}>
            Specific period
          </button>
        </li>
      </ul>

      <section>
        {#if activeTab !== 'last'}
          <div class="calendar-container">
            <p class="subtitle">From Date</p>
            <span class="calendar-icon">
              <input
                data-type="specific"
                type="date"
                id="selectStartDateElement"
              />
            </span>
          </div>
        {/if}

        {#if activeTab === 'period'}
          <div class="calendar-container">
            <p class="subtitle">To Date</p>
            <span class="calendar-icon">
              <input
                data-type="specific"
                type="date"
                id="selectEndDateElement"
              />
            </span>
          </div>
        {/if}

        <div>
          <p class="subtitle">Select Group Name</p>
          <select class="select"
            ><option value="">All Markets</option>
            {#each markets as market}
              <option value={market.id}>{market.name}</option>
            {/each}
          </select>
        </div>

        <div>
          <button class="emp-btn" on:click={() => downloadFile('XML')}
            >XML</button
          >
          <button class="emp-btn" on:click={() => downloadFile('JSON')}
            >JSON</button
          >
          <button class="emp-btn excel" on:click={() => downloadFile('EXCEL')}
            >EXCEL</button
          >
        </div>
      </section>
    </div>
  </div>
</main>

<style>
</style>
