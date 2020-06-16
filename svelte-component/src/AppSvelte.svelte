<script>
  import { onMount } from "svelte";
  import Switch from "./Components/Switch.svelte";
  import Card from "./Components/Card.svelte";
  import Select from "svelte-select";

  let exchangeRate = false,
    error = "",
    items,
    currencies = [],
    countries = [],
    countriesLeft,
    countriesRight,
    currrenciesLeft,
    currrenciesRight,
    inputValue = "",
    output = "0",
    timer;

  $: !!inputValue &&
    !!currrenciesLeft &&
    !!currrenciesRight &&
    exchangeRate &&
    convert();

  $: !!inputValue &&
    !!currrenciesLeft &&
    !!currrenciesRight &&
    !!countriesLeft &&
    !!countriesRight &&
    !!output &&
    !exchangeRate &&
    transformConvert();

  function switchMode() {
    exchangeRate = !exchangeRate;
  }

  onMount(() => {
    fetch("http://localhost:4321/currencies")
      .then((response) => response.json())
      .then((data) => {
        items = data;
        countries = items["countries"].map((i) => {
          return { value: i.code, label: i.name };
        });
        currencies = items["currencies"].map((i) => {
          return { value: i.code, label: i.name };
        });
      })
      .catch((e) => (error = "Error fetching data"));
  });

  function debounce(v) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      inputValue = v;
    }, 750);
  }

  function convert() {
    fetch(
      `http://localhost:4321/convert/${inputValue}${currrenciesLeft}/${currrenciesRight}`
    )
      .then((response) => response.json())
      .then((data) => {
        console.log("data.data.", data.data);
        const { Value, Currency } = data.data.new_price;
        output = `${Value.toFixed(2)} ${Currency}`;
      });
  }

  function transformConvert() {
    fetch(
      `http://localhost:4321/fair/${inputValue}${currrenciesLeft}%40${countriesLeft.toUpperCase()}/${countriesRight.toUpperCase()}/${currrenciesRight}`
    )
      .then((response) => response.json())
      .then((data) => {
        const { Value, Currency, CountryCode } = data.data.new_local_price;
        output = `${Value.toFixed(2)} ${Currency} in ${CountryCode}`;
      });
  }

  function handleSelect(a, e) {
    switch (a) {
      case "currrenciesLeft":
        currrenciesLeft = e.detail.value;
        break;
      case "currrenciesRight":
        currrenciesRight = e.detail.value;
        break;
      case "countriesRight":
        countriesRight = e.detail.value;
        break;
      case "countriesLeft":
        countriesLeft = e.detail.value;
        break;
    }
  }
</script>

<style>
  .wrapper {
    padding: 15px;
  }
  .switch {
    display: flex;
    justify-content: center;
    padding: 25px;
  }

  .switch span {
    margin: 0 15px;
  }

  .container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 110px;
  }

  .equal {
    font-size: 80px;
  }

  .top {
    display: flex;
    align-items: center;
  }

  .top > :first-child {
    height: 42px;
    margin: 0 10px 0 0;
    flex: 1;
    border-radius: 3px;
  }

  .top p {
    padding: 10px;
    border: 1px solid #d8dbdf;
    color: #3f4f5f;
  }

  :global(.top div, .top input) {
    flex: 1;
    justify-content: center;
    align-items: center;
    margin-bottom: 0;
  }

  .top input {
    border: 1px solid #d8dbdf;
  }

  .bottom {
    margin-top: 15px;
  }

  :global(.bottom input) {
    margin-bottom: 0;
  }

  @media (max-width: 800px) {
    .container,
    .equal {
      display: block;
    }

    .equal {
      text-align: center;
    }
  }
</style>

{#if !!items}
  <div class="wrapper">
    <div class="switch">
      <span>fair price</span>
      <Switch on:handleSwitch={switchMode} isActive={exchangeRate} />
      <span>exchange rate</span>
    </div>
    <div class="container">
      <Card>
        <div slot="top" class="top">
          <input
            type="number"
            placeholder="introduce amount"
            on:input={({ target: { value } }) => debounce(value)}
            min="0" />
          <Select
            items={currencies}
            placeholder="select currency"
            on:select={(e) => handleSelect('currrenciesLeft', e)} />
        </div>
        <div hidden={exchangeRate} slot="bottom" class="bottom">
          <Select
            items={countries}
            placeholder="select country"
            on:select={(e) => handleSelect('countriesLeft', e)} />
        </div>
      </Card>
      <span class="equal">{'='}</span>
      <Card>
        <div slot="top" class="top">
          <p>{!!inputValue ? output : 0}</p>
          <Select
            items={currencies}
            placeholder="select currency"
            on:select={(e) => handleSelect('currrenciesRight', e)} />
        </div>
        <div hidden={exchangeRate} slot="bottom" class="bottom">
          <Select
            items={countries}
            placeholder="select country"
            on:select={(e) => handleSelect('countriesRight', e)} />
        </div>
      </Card>
    </div>
  </div>
{:else}
  <p>{error}</p>
{/if}
