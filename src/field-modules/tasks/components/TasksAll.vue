<template lang="html">
  <farm-main :space="['0', '1rem']">
    <farm-tiles :columns="[1, 2, 3]" :space="[0, '1rem']" :dividers="[true, false]">
      <farm-card
        v-if="logs.length < 1">
        <farm-stack :space="'0.25rem'">
          <h4>{{ $t('Let\'s Get Started!')}}</h4>
          <p>
            {{ $t('You don\'t have any logs to display yet. Logs are records of events') }}
            {{ $t('in farmOS. You can add some by clicking the') }}
            <icon-add-circle class="inline-svg"/>
            {{ $t('icon below, or you can get uncompleted tasks from the server by') }}
            {{ $t('clicking the') }} <icon-cloud-upload class="inline-svg"/> {{ $t('icon above.') }}
          </p>
        </farm-stack>
      </farm-card>
      <farm-card
        v-for="(log, i) in logs.filter(passesFilters)"
        :key="`card-${i}`">
        <router-link :to="{ path: `/tasks/${log.localID}` }">
          <farm-stack :space="'0.25rem'">

            <div class="card-row-1">
              <icon-assignment-done
                class="assignment done"
                v-if="log.done"/>
              <icon-assignment
                class="assignment"
                v-if="!log.done && (log.timestamp * 1000 > new Date().valueOf())"/>
              <icon-assignment-late
                class="assignment late"
                v-if="!log.done && (log.timestamp * 1000 < new Date().valueOf())"/>
              <div class="log-name">
                <h5>{{log.name}}</h5>
              </div>
              <icon-cloud-upload v-if="isUnsynced(log)" class="sync-status"/>
              <icon-cloud-done v-else class="sync-status"/>
            </div>

            <div class="card-row-2">
              <p>{{parseNotes(log.notes)}}</p>
            </div>

            <div class="card-row-3">
              <div class="date-and-type">
                <span class="log-type">{{$t(logTypes[log.type].label).toUpperCase()}}</span>
                <span>{{showDate(log.timestamp)}}</span>
              </div>
              <div class="tags">
                <span
                  v-for="(area, i) in mapTidsToAreas(log)"
                  class="tag tag-area"
                  :key="`area-${i}`">
                  {{area.name}}
                </span>
                <span
                  v-for="(asset, i) in mapIdsToAssets(log)"
                  class="tag tag-asset"
                  :key="`asset-${i}`">
                  {{asset.name}}
                </span>
              </div>
            </div>

          </farm-stack>
        </router-link>
      </farm-card>
    </farm-tiles>

    <div class="add-circle" @click="startNewLog">
      <div class="background-circle">
      </div>
      <icon-add-circle/>
    </div>

  </farm-main>

</template>

<script>
const { parseNotes } = window.farmOS.utils;
const { isUnsynced } = window.farmOS.utils.farmLog;

export default {
  name: 'TasksAll',
  props: [
    'logTypes',
    'logs',
    'userId',
    'assets',
    'logDisplayFilters',
    'areas',
  ],
  methods: {
    showDate(unixTimestamp) {
      if (Number.isNaN(Number(unixTimestamp))) {
        return 'No Date Provided';
      }
      const date = new Date(unixTimestamp * 1000);
      const opts = { month: 'short', day: 'numeric', year: 'numeric' };
      return date.toLocaleDateString(undefined, opts);
    },
    // Pass in a log and get back an array of the areas attached to that log
    mapTidsToAreas(log) {
      if (log.area && this.areas.length > 0) {
        return log.area.map(a1 => this.areas.find(a2 => +a2.tid === +a1.id));
      }
      return [];
    },
    // Pass in a log and get back an array of the areas attached to that log
    mapIdsToAssets(log) {
      if (log.asset && this.assets.length > 0) {
        return log.asset.map(a1 => this.assets.find(a2 => +a2.id === +a1.id));
      }
      return [];
    },
    passesFilters(log) {
      const passesTypeFilter = !this.logDisplayFilters.excludedTypes.includes(log.type);
      const passesCategoryFilter = () => {
        const logHasNoCats = log.log_category === null
          || log.log_category.length === 0;
        const noCatsFilterIsSelected = this.logDisplayFilters.excludedCategories.includes(-1);
        if (logHasNoCats && noCatsFilterIsSelected) {
          return false;
        }
        if (log.log_category !== null) {
          return !log.log_category.some(cat => (
            this.logDisplayFilters.excludedCategories.some(exCat => +exCat === +cat.id)
          ));
        }
        return true;
      };
      const passesDateFilter = () => {
        if (Number.isNaN(Number(log.timestamp))) {
          return true;
        }
        const filter = this.logDisplayFilters.date;
        const d = new Date();
        let dateLimit;
        if (filter === 'TODAY') {
          d.setDate(d.getDate() - 1);
          dateLimit = Math.floor(d.valueOf() / 1000);
        }
        if (filter === 'THIS_WEEK') {
          d.setDate(d.getDate() - 7);
          dateLimit = Math.floor(d.valueOf() / 1000);
        }
        if (filter === 'THIS_MONTH') {
          d.setMonth(d.getMonth() - 1);
          dateLimit = Math.floor(d.valueOf() / 1000);
        }
        if (filter === 'THIS_YEAR') {
          d.setYear(d.getYear() - 1);
          dateLimit = Math.floor(d.valueOf() / 1000);
        }
        if (filter === 'ALL_TIME') {
          dateLimit = 0;
        }
        return log.timestamp > dateLimit;
      };

      if (passesTypeFilter && passesCategoryFilter() && passesDateFilter()) {
        return true;
      }
      return false;
    },
    startNewLog() {
      this.initializeLog({ done: true })
        .then(id => this.$router.push({ path: `/tasks/${id}` }));
    },
    parseNotes,
    isUnsynced,
  },
};

</script>

<style scoped>
  .inline-svg {
    height: 1rem;
  }

  a:hover {
    text-decoration: none;
  }

  a {
    color: var(--dark);
  }

  p, h5 {
    margin: 0
  }

  .card-row-1 {
    display: flex;
    flex-flow: row nowrap;
  }

  .card-row-1 h5 {
    font-weight: 700;
  }

  .card-row-2 {
    display: flex;
    flex-flow: row nowrap;
  }

  .card-row-3 {
    display: flex;
    flex-flow: row nowrap;
    justify-content: space-between;
  }

  .assignment {
    flex: 0 0 auto;
  }

  .assignment.late {
    fill: var(--warning);
  }

  .log-name {
    flex: 3 1 auto;
    margin-left: 1rem;
  }

  .sync-status {
    flex: 0 0 auto;
  }

  .date-and-type {
    display: flex;
    flex-flow: column;
    justify-content: flex-end;
    flex: 0 0 auto;
  }

  .date-and-type span {
    margin-bottom: .25rem;
    padding: .125rem 0;
  }

  .log-type {
    font-size: .75rem;
  }

  .tags {
    display: flex;
    justify-content: flex-end;
    align-items: flex-end;
    flex-flow: row wrap;
    flex: 0 1 auto;
  }

  .tag {
    margin-left: .25rem;
    margin-bottom: .25rem;
    padding: .125rem;
    border-radius: .25rem;
  }

  .tag-area {
    border: 1px solid var(--blue);
    background-color: var(--accent-blue);
  }

  .tag-asset {
    border: 1px solid var(--green);
    background-color: var(--accent-green);
  }

  @media (min-width: 576px) {
    .card-group .card {
      flex: 0 0 576px;
    }
  }

  .add-circle {
    position: fixed;
    bottom: 1.5rem;
    right: 1.5rem;
  }

  .add-circle svg {
    fill: var(--primary);
    height: 4.5rem;
    width: 4.5rem;
  }

  .background-circle {
    height: 3.75rem;
    width: 3.75rem;
    border-radius: 2rem;
    background-color: var(--accent-primary);
    box-shadow: var(--shadow-strong);
    position: absolute;
    top: .375rem;
    left: .375rem;
    z-index: -10;
  }

</style>