<script>
  import { onMount } from 'svelte';
  import SvelteMarkdown from 'svelte-markdown';
  import { DateTime } from 'luxon';

  let loading = true;
  let agenda_list = {};
  let categories = [];
  let venues = [];

  let active_tab = '';
  let active_venue = '';

  async function loadData() {
    loading = true;

    try {
      const response = await fetch(
        'https://events.startupmission.in/api/event/sep/agenda/venue'
      );

      if (!response.ok) {
        throw new Error(`Agenda API returned ${response.status}`);
      }

      const data = await response.json();

      agenda_list = data.agenda ?? {};
      categories = data.categories ?? [];
      venues = data.venues ?? [];

      const availableDates = dates();

      const today = DateTime
        .now()
        .setZone('Asia/Kolkata')
        .toFormat('LLL dd, yyyy');

      active_tab = availableDates.includes(today)
        ? today
        : availableDates[0] ?? '';

      setDefaultVenue();

    } catch (error) {
      console.error('Failed to load agenda:', error);
    } finally {
      loading = false;
    }
  }

  onMount(loadData);


  /* ----------------------------------------
     DATE
  ---------------------------------------- */

  function dates() {
    return Object.keys(agenda_list);
  }

  function parseAgendaDate(date) {
    return DateTime.fromFormat(date, 'LLL dd, yyyy');
  }

  function dateFormat(date, format) {
    const d = parseAgendaDate(date);

    if (!d.isValid) {
      return date;
    }

    return d.toFormat(format);
  }

  function selectTab(date) {
    active_tab = date;
    setDefaultVenue();
  }


  /* ----------------------------------------
     VENUE
  ---------------------------------------- */

  function getActiveDateVenues() {
    if (!active_tab || !agenda_list[active_tab]) {
      return [];
    }

    return Object.keys(agenda_list[active_tab]);
  }

  function setDefaultVenue() {
    const availableVenues = getActiveDateVenues();

    if (
      active_venue &&
      availableVenues.includes(active_venue)
    ) {
      return;
    }

    active_venue = availableVenues[0] ?? '';
  }

  function selectVenue(venue) {
    active_venue = venue;
  }


  /* ----------------------------------------
     TIME
  ---------------------------------------- */

  function parseTime(date) {
    return DateTime.fromSQL(date, {
      zone: 'Asia/Kolkata'
    });
  }

  function timeOnly(date) {
    const d = parseTime(date);

    if (!d.isValid) {
      return '';
    }

    return d.toFormat('h.mm a');
  }

  function timeRange(start, end) {
    const s = parseTime(start);
    const e = parseTime(end);

    if (!s.isValid) {
      return '';
    }

    if (!e.isValid) {
      return s.toFormat('h.mm a');
    }

    return `${s.toFormat('h.mm a')} – ${e.toFormat('h.mm a')}`;
  }


  /* ----------------------------------------
     CURRENT AGENDA
  ---------------------------------------- */

  function currentAgenda() {
    if (
      !active_tab ||
      !active_venue ||
      !agenda_list[active_tab]
    ) {
      return [];
    }

    return agenda_list[active_tab][active_venue] ?? [];
  }


  /* ----------------------------------------
     MORNING / AFTERNOON GROUPING
  ---------------------------------------- */

  function getPeriod(startTime) {
    const d = parseTime(startTime);

    if (!d.isValid) {
      return 'Morning';
    }

    return d.hour < 13 ? 'Morning' : 'Afternoon';
  }

  function groupedAgenda() {
    const agenda = currentAgenda();

    return {
      Morning: agenda.filter(
        (item) => getPeriod(item.start_time) === 'Morning'
      ),

      Afternoon: agenda.filter(
        (item) => getPeriod(item.start_time) === 'Afternoon'
      )
    };
  }


  /* ----------------------------------------
     PERIOD DESCRIPTION
  ---------------------------------------- */

  function periodDescription(period) {
    if (period === 'Morning') {
      return 'Registration, inauguration & sessions';
    }

    return 'Conversations, networking & ecosystem support';
  }


  /* ----------------------------------------
     FEATURED SESSION
  ---------------------------------------- */

  function isFeatured(item) {
    const text = [
      item.name,
      item.category
    ]
      .filter(Boolean)
      .join(' ')
      .toLowerCase();

    const featuredTerms = [
      'inauguration',
      'keynote',
      'launch',
      'panel',
      'fireside',
      'plenary',
      'special address'
    ];

    return featuredTerms.some(
      (term) => text.includes(term)
    );
  }


  /* ----------------------------------------
     SESSION DETAILS
  ---------------------------------------- */

  function hasSpeakers(speakers) {
    if (!speakers) {
      return false;
    }

    return Object.values(speakers).some(
      (list) => Array.isArray(list) && list.length
    );
  }

  function hasDetails(item) {
    return Boolean(
      item.description ||
      hasSpeakers(item.speakers)
    );
  }


  /* ----------------------------------------
     SPEAKER HELPERS
  ---------------------------------------- */

  function speakerRole(speaker) {
    return [
      speaker.designation,
      speaker.organisation
    ]
      .filter(Boolean)
      .join(', ');
  }
</script>


<section
  id="agenda"
  class=" section-white agenda-section"
>

  <div class="max-w-7xl mx-auto px-5 md:px-8 lg:px-10">


    <!-- ======================================
         HEADING
    ======================================= -->

    <div class="agenda-heading reveal">

      <div>

        <span class="section-index">
          07
        </span>

        <p class="section-eyebrow">
          Programme
        </p>

        <h2 class="section-title">
          Agenda
        </h2>

      </div>


      {#if active_tab}

        <div class="agenda-date">

          <span>
            {dateFormat(active_tab, 'cccc')}
          </span>

          <strong>
            {dateFormat(active_tab, 'dd')}
          </strong>

          <p>
            {dateFormat(active_tab, 'LLLL yyyy')}
          </p>

        </div>

      {/if}

    </div>



    <!-- ======================================
         LOADING
    ======================================= -->

    {#if loading}

      <div class="agenda-loading mt-16">

        <div class="agenda-loading-line"></div>
        <div class="agenda-loading-line"></div>
        <div class="agenda-loading-line"></div>
        <div class="agenda-loading-line"></div>

      </div>


    {:else if dates().length === 0}

      <div class="agenda-empty mt-16">
        Programme details will be announced soon.
      </div>


    {:else}


      <!-- ======================================
           DATE TABS
           Only visible when multiple dates exist
      ======================================= -->

      {#if dates().length > 1}

        <div class="agenda-tabs mt-10">

          {#each dates() as date}

            <button
              type="button"
              class:active={date === active_tab}
              on:click={() => selectTab(date)}
            >
              <span>
                {dateFormat(date, 'ccc')}
              </span>

              {dateFormat(date, 'dd LLL')}
            </button>

          {/each}

        </div>

      {/if}



      <!-- ======================================
           VENUE TABS
           Only visible when multiple venues exist
      ======================================= -->

      {#if getActiveDateVenues().length > 1}

        <div class="agenda-venue-tabs mt-5">

          {#each getActiveDateVenues() as venue}

            <button
              type="button"
              class:active={venue === active_venue}
              on:click={() => selectVenue(venue)}
            >
              {venue}
            </button>

          {/each}

        </div>

      {/if}



      <!-- ======================================
           AGENDA
      ======================================= -->

      <div class="agenda-container mt-16 reveal">


        {#each Object.entries(groupedAgenda()) as [period, sessions]}


          {#if sessions.length}


            <div class="agenda-block">


              <!-- PERIOD -->

              <div class="agenda-period">

                <span>
                  {period}
                </span>

                <p>
                  {periodDescription(period)}
                </p>

              </div>



              <!-- EVENTS -->

              <div class="agenda-events">


                {#each sessions as item}


                  <div
                    class="agenda-event"
                    class:agenda-event-featured={isFeatured(item)}
                  >


                    <!-- TIME -->

                    <time>
                      {timeOnly(item.start_time)}
                    </time>



                    <!-- CONTENT -->

                    <div>


                      {#if item.category}

                        <span class="agenda-category">
                          {item.category}
                        </span>

                      {/if}



                      <h3>
                        {item.name}
                      </h3>



                      <!-- Optional simple time range -->

                      {#if item.end_time}

                        <p class="agenda-duration">
                          {timeRange(
                            item.start_time,
                            item.end_time
                          )}
                        </p>

                      {/if}



                      <!-- ==================================
                           EXPANDABLE DETAILS
                      =================================== -->

                      {#if hasDetails(item)}

                        <details class="agenda-details">

                          <summary>

                            View session details

                            <span>
                              ＋
                            </span>

                          </summary>


                          <div class="agenda-details-body">


                            <!-- DESCRIPTION -->

                            {#if item.description}

                              <div class="agenda-markdown">
                                <SvelteMarkdown
                                  source={item.description}
                                />
                              </div>

                            {/if}



                            <!-- SPEAKERS -->

                            {#if hasSpeakers(item.speakers)}

                              <div class="agenda-speakers">


                                {#each Object.entries(item.speakers) as [speakerCategory, speakerList]}


                                  {#if speakerList?.length}

                                    <div class="agenda-speaker-group">


                                      {#if speakerCategory}

                                        <span class="agenda-speaker-category">
                                          {speakerCategory}
                                        </span>

                                      {/if}



                                      {#each speakerList as speaker}

                                        <div class="agenda-speaker">


                                          {#if speaker.photo}

                                            <img
                                              src={speaker.photo}
                                              alt={speaker.name}
                                              loading="lazy"
                                            />

                                          {/if}



                                          <div>

                                            {#if speaker.linkedin}

                                              <a
                                                href={speaker.linkedin}
                                                target="_blank"
                                                rel="noopener noreferrer"
                                              >
                                                {speaker.name}
                                              </a>

                                            {:else}

                                              <strong>
                                                {speaker.name}
                                              </strong>

                                            {/if}


                                            {#if speakerRole(speaker)}

                                              <p>
                                                {speakerRole(speaker)}
                                              </p>

                                            {/if}

                                          </div>

                                        </div>

                                      {/each}


                                    </div>

                                  {/if}


                                {/each}

                              </div>

                            {/if}


                          </div>

                        </details>

                      {/if}


                    </div>

                  </div>


                {/each}


              </div>

            </div>


          {/if}


        {/each}


      </div>

    {/if}

  </div>

</section>