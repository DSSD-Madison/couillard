<script>
  import { onMount } from 'svelte';
  import { totalData, panelName } from '../stores';
  import { TOTALS_DERIVATIONS, FORMATTING_LOCALE, CURRENCY } from '../conversions';

  let totals = 0;
  onMount(() => {
    totalData.subscribe(async promise => {
      totals = await promise;
    });
  });

  const SIG_CHARS = 5;
  
  const compactNumberFormatter = Intl.NumberFormat(FORMATTING_LOCALE, {
    notation: 'compact',
    maximumSignificantDigits: SIG_CHARS - 1,
  });
  const siNumberFormatter = Intl.NumberFormat(FORMATTING_LOCALE, {
    notation: 'compact',
    maximumSignificantDigits: SIG_CHARS,
  })
  const moneyFormatterNoDP = Intl.NumberFormat(FORMATTING_LOCALE, {
    style: 'currency',
    currency: CURRENCY,
    maximumFractionDigits: 0,
    minimumFractionDigits: 0,
  });
  const moneyFormatter = Intl.NumberFormat(FORMATTING_LOCALE, {
    style: 'currency',
    currency: CURRENCY,
  });
  function pickSiPrefix(num, just) {
    let everything = siNumberFormatter.format(num).replace("K", "k").replace("B", "G");
    let prefixless = !(/[A-Za-z]$/.test(everything));
    switch (just) {
      case "prefix":
        if (prefixless) return "";
        return everything.slice(-1);
      case "number":
        if (prefixless) return everything;
        return everything.slice(0, -1);
      default:
        console.warn("Returning undefined SI prefix, withOrWithout must be prefix or number.");
    }
  }
  function fmtNum(num, unitType, forceHigherPrecision = false) {
    switch (unitType) {
      case "money":
        return (
          forceHigherPrecision ? moneyFormatter : moneyFormatterNoDP
        ).format(num);
      case "si":
        return pickSiPrefix(num, "number");
      case "expanded":
        return forceHigherPrecision ? num.toLocaleString() : Math.floor(num).toLocaleString();
      case "simple":
      default:
        let result = compactNumberFormatter.format(num);
        if (unitType !== "simple") {
          console.warn(`Unknown unit type ${unitType}, defaulting to simple.`);
        }
        return result;
    }
  }
  function fmtUnit(num, unit, unitType, shortForm = false) {
    switch (unitType) {
      case "money":
        if (shortForm) return "";
        return unit;
      case "si":
        let siPrefix = pickSiPrefix(num, "prefix");
        return `${siPrefix}${unit}`;
      case "simple":
      case "expanded":
        return unit;
      default:
        console.warn(`Unknown unit type ${unitType}, defaulting to simple.`);
        return unit;
    }
  }
</script>

<div class="stats-row">
  {#if totals == 0}
  No data has been tracked for this array yet
  {:else}
    {#if $panelName !== "all"}
      The {$panelName} Array
    {:else}
      Solar arrays in Deerfield
    {/if}...
  <div class="stats-wrapper">
    {#each TOTALS_DERIVATIONS as [conv, unit, unitType, unitSuffix, showDesc]}
      <div class="statistic">
          <p class="large">{fmtNum(conv * totals, unitType)}</p>
          <p class="medium statistic-detail">
            {#if unit === "" && unitSuffix === ""}
              <div class="invisible-placeholder">X</div>
            {:else}
              {fmtUnit(conv * totals, unit, unitType)} {unitSuffix}
            {/if}
          </p>
          {#if showDesc}
            <p class="small statistic-detail">
              *Based on {fmtNum(conv, unitType, true)}
              {fmtUnit(conv, unit, unitType, true)} {unitSuffix.toLowerCase()} per kWh
            </p>
          {/if}
      </div>
    {/each}
  </div>
  <p class="tiny">Totals are calculated from real-time tracking systems as well as historic utility data. The graph below only shows the former.</p>
  {/if}
</div>

<style>
/* UNUSED .info-title {
  font-size: 36px;
  font-weight: bolder;
  color: black;
} */


.stats-row {
  margin-top: 0px;
  color: white;
  font-size: x-large;
  background-color: var(--couillard-blue-color);
  text-align: center;
  padding: 20px;
  font-size: 24px;
}

.invisible-placeholder {
  visibility: hidden;
}

.stats-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  width: 100%;
}

.statistic {
  text-align: center;
  color: white;
  font-weight: bold;
  width: 33%;
}

@media(max-width: 760px) {
  .statistic {
    width: 100%;
  }
}

.large {
  font-size: 4rem;
  margin: 0%;
  font-weight: bold;
}

.medium {
  font-size: 2rem;
  margin: 0%;
}

.small {
  font-size: 1rem;
  margin: 0%;
}

.tiny {
  font-size: 0.7rem;
  margin: 0%;
}

</style>
