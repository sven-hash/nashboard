<script context="module">
  export async function load({ fetch }) {
    // to prevent the constant loading, store data in
    // svelte store, then only fetch if svelte store is empty
    const response = await fetch("/api/data/get/events");
    const data = await response.json();

    const monthlyResponse = await fetch("/api/data/get/monthly");
    const monthlyData = await monthlyResponse.json();

    const yearlyResponse = await fetch("/api/data/get/yearly");
    const yearlyData = await yearlyResponse.json();

    return {
      status: response.status,
      props: {
        data: data,
        monthlyData: monthlyData,
        yearlyData: yearlyData,
      },
    };
  }
</script>

<script>
  export let data = {};
  export let monthlyData = {};
  export let yearlyData = {};
  import _ from "underscore";
  import Activity from "$lib/Activity.svelte";
  import Pie from "$lib/Pie.svelte";
  import Title from "$lib/Title.svelte";
  import SingleNumber from "$lib/SingleNumber.svelte";
  import Relay from "$lib/Relay.svelte";
  import MiddleText from "$lib/MiddleText.svelte";
  import TweetList from "$lib/TweetList.svelte";
  import LinkOut from "$lib/LinkOut.svelte";
  import Month from "$lib/Month.svelte";
  import Year from "$lib/Year.svelte";

  const allTweets = data.events;
  const shortListAmount = 8;
  const longListAmount = Math.min(30, allTweets.length);

  let tweets = allTweets.slice(0, shortListAmount);
  let extendedTweets = allTweets.slice(shortListAmount, longListAmount);

  let allFoundIn = [];
  for (let i = 0; i < allTweets.length; i++) {
    // shuffle the foundIn array just for the aesthetic
    allFoundIn.push(_.shuffle(data.where[i].existsIn));
  }
  let foundIn = allFoundIn.slice(0, longListAmount);
</script>

<!-- if fetch error -->
{#if tweets.length == 0}
  <MiddleText
    message={"oops, something went wrong while fetching the data, refresh page"}
  />
{:else}
  <div class="p-2 space-y-1 max-w-lg mx-auto xl:hidden py-7">
    <div class="space-y-1">
      <Title />
      <LinkOut />
      <div class="sm:flex space-y-1 sm:space-y-0 sm:space-x-1">
        <div class="flex-1">
          <SingleNumber number={data.eventCount} label={"EVENT COUNT 24H"} />
        </div>
        <div class="flex-1">
          <SingleNumber
            number={data.uniquePubkeys}
            label={"UNIQUE PUBKEYS 24H"}
          />
        </div>
      </div>
      <Activity networkActivity={data.utc} />
    </div>

    <TweetList {tweets} {extendedTweets} {foundIn} />

    <div class="space-y-1">
      <Pie kinds={data.kinds} />
      <Relay relayData={data.relays} />
      <Month {monthlyData} />
      <Year {yearlyData} />
    </div>
  </div>

  <div class="hidden xl:flex xl:flex-col mx-auto max-w-7xl py-7">
    <Title />
    <LinkOut />
    <div class="flex space-x-1 items-start">
      <div class="space-y-1 shrink-0">
        <SingleNumber number={data.eventCount} label={"EVENT COUNT 24H"} />
        <SingleNumber
          number={data.uniquePubkeys}
          label={"UNIQUE PUBKEYS 24H"}
        />
      </div>

      <TweetList {tweets} {extendedTweets} {foundIn} />

      <div class="space-y-1 basis-1/2">
        <Activity networkActivity={data.utc} />
        <Pie kinds={data.kinds} />
        <Relay relayData={data.relays} />
        <Month {monthlyData} />
        <Year {yearlyData} />
      </div>
    </div>
  </div>
{/if}
