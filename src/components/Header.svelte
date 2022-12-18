<script lang="ts">
  import logo from '../assets/logo.svg';
  import logout from '../assets/log-out.svg';

  const userName = localStorage.getItem('userName');
</script>

<header>
  <div class="header-wrapper">
    <img class="logo" src={logo} alt="logo" />

    {#if userName}
      <p class="user-name-field">{userName}</p>
    {/if}

    <a class="log-out-link" href="https://www.energymarketprice.com/home/logout"
      ><img src={logout} class="log-out-icon" alt="logout icon" /> Logout</a
    >
  </div>
</header>

<style>
  header {
    height: 84px;
    width: 100vw;
    background-color: white;
    display: flex;
    justify-content: center;
    font-size: 14px;
    position: absolute;
    top: 0;
    left: 0;
    border-bottom: 1px solid #bec4cc;
  }

  .header-wrapper {
    width: 80%;
    max-width: 1200px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .logo {
    margin-right: auto;
  }

  .log-out-icon {
    margin: 0 10px 0 30px;
  }

  .log-out-link {
    text-decoration: none;
    color: black;
    display: flex;
    align-items: center;
  }

  .user-name-field {
    color: #818890;
  }

  @media (max-width: 700px) {
    .header-wrapper {
      flex-direction: column;
    }
    .logo {
      width: 280px;
      margin: 20px;
    }

    header {
      height: 100px;
      padding-bottom: 10px;
    }
  }
</style>
