<script>
  import { onMount } from 'svelte';
  import { getAddress } from '@ethersproject/address';
  import { setDefaults as setToast, toast } from 'bulma-toast';

  let input = null;
  let faucetInfo = {
    account: '0x0000000000000000000000000000000000000000',
    network: 'testnet',
    payout: 1,
    symbol: 'ETH',
    hcaptcha_sitekey: '',
  };

  let mounted = false;
  let hcaptchaLoaded = false;

  onMount(async () => {
    const res = await fetch('/api/info');
    faucetInfo = await res.json();
    mounted = true;
  });

  window.hcaptchaOnLoad = () => {
    hcaptchaLoaded = true;
  };

  $: document.title = `${faucetInfo.symbol} ${capitalize(
    faucetInfo.network,
  )} Faucet`;

  let widgetID;
  $: if (mounted && hcaptchaLoaded) {
    widgetID = window.hcaptcha.render('hcaptcha', {
      sitekey: faucetInfo.hcaptcha_sitekey,
    });
  }

  setToast({
    position: 'bottom-center',
    dismissible: true,
    pauseOnHover: true,
    closeOnClick: false,
    animate: { in: 'fadeIn', out: 'fadeOut' },
  });

  async function handleRequest() {
    let address = input;
    if (address === null) {
      toast({ message: 'input required', type: 'is-warning' });
      return;
    }

    try {
      address = getAddress(address);
    } catch (error) {
      toast({ message: error.reason, type: 'is-warning' });
      return;
    }

    try {
      let headers = {
        'Content-Type': 'application/json',
      };

      if (hcaptchaLoaded) {
        const { response } = await window.hcaptcha.execute(widgetID, {
          async: true,
        });
        headers['h-captcha-response'] = response;
      }

      const res = await fetch('/api/claim', {
        method: 'POST',
        headers,
        body: JSON.stringify({
          address,
        }),
      });

      let { msg } = await res.json();
      let type = res.ok ? 'is-success' : 'is-warning';
      toast({ message: msg, type });
    } catch (err) {
      console.error(err);
    }
  }

  function capitalize(str) {
    const lower = str.toLowerCase();
    return str.charAt(0).toUpperCase() + lower.slice(1);
  }
</script>

<svelte:head>
  {#if mounted && faucetInfo.hcaptcha_sitekey}
    <script
      src="https://hcaptcha.com/1/api.js?onload=hcaptchaOnLoad&render=explicit"
      async
      defer
    ></script>
  {/if}
</svelte:head>

<main>
  <section class="hero is-info is-fullheight">
    <div class="hero-head">
      <nav class="navbar">
        <div class="container">
          <div class="navbar-brand">
            <a class="navbar-item" href="../..">
              <span class="icon">
                <i class="fa fa-bath" />
              </span>
              <span><b>{faucetInfo.symbol} Faucet</b></span>
            </a>
          </div>
        </div>
      </nav>
    </div>

    <div class="hero-body">
      <div class="container">
        <div class="columns is-vcentered">
          <!-- Left Column - Protocol Info -->
          <div class="column is-5 left-column">
            <img src="/logo.png" alt="Over Protocol Logo" class="protocol-logo" />
            <div class="protocol-info">
              <h2 class="title is-4">Faucet for Dolphin</h2>
              <div class="links-section">
                <a href="https://docs.over.network" class="protocol-link" target="_blank" rel="noopener noreferrer">
                  <span class="icon"><i class="fa fa-book"></i></span>
                  <span>Documentation</span>
                </a>
                <a href="https://discord.com/invite/overprotocol" class="protocol-link" target="_blank" rel="noopener noreferrer">
                  <span class="icon"><i class="fa fa-comments"></i></span>
                  <span>Discord Community</span>
                </a>
                <a href="https://github.com/overprotocol" class="protocol-link" target="_blank" rel="noopener noreferrer">
                  <span class="icon"><i class="fa fa-github"></i></span>
                  <span>GitHub</span>
                </a>
              </div>
            </div>
          </div>

          <!-- Right Column - Faucet Interface -->
          <div class="column is-7">
            <div class="faucet-container">
              <h1 class="title">
                Receive {faucetInfo.payout}
                {faucetInfo.symbol} per request
              </h1>
              <h2 class="subtitle">
                Serving from {faucetInfo.account}
              </h2>
              <div id="hcaptcha" data-size="invisible"></div>
              <div class="box">
                <div class="field is-grouped">
                  <p class="control is-expanded">
                    <input
                      bind:value={input}
                      class="input is-rounded"
                      type="text"
                      placeholder="Enter your address"
                    />
                  </p>
                  <p class="control">
                    <button
                      on:click={handleRequest}
                      class="button is-primary is-rounded"
                    >
                      Request
                    </button>
                  </p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</main>

<style>
  .hero.is-info {
    background: linear-gradient(135deg, #1a1a1a 0%, #363636 100%);
  }

  .hero .subtitle {
    padding: 1.5rem 0;
    line-height: 1.5;
  }

  .box {
    border-radius: 19px;
    background: rgba(255, 255, 255, 0.9);
    padding: 1.5rem;
  }

  .left-column {
    padding: 2rem;
    background: rgba(255, 255, 255, 0.9);
    border-radius: 15px;
  }

  .protocol-logo {
    max-width: 200px;
    margin-bottom: 2rem;
  }

  .protocol-info {
    color: #1a1a1a;
  }

  .protocol-info h2 {
    color: #1a1a1a !important;
  }

  .links-section {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin-top: 2rem;
  }

  .protocol-link {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: #1a1a1a;
    padding: 0.5rem;
    border-radius: 8px;
    transition: background-color 0.3s ease;
  }

  .protocol-link:hover {
    background-color: rgba(0, 0, 0, 0.1);
    color: #1a1a1a;
  }

  .faucet-container {
    padding: 2rem;
    color: white;
  }

  .faucet-container .title,
  .faucet-container .subtitle {
    color: white;
  }

  .input.is-rounded {
    box-shadow: none;
    border: 2px solid #eee;
  }

  .input.is-rounded:focus {
    border-color: #F5773E;
  }

  .button.is-primary {
    background-color: #F5773E;
    border-color: transparent;
  }

  .button.is-primary:hover {
    background-color: #e66a31;
  }
</style>
