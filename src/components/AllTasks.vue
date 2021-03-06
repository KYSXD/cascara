<template>
  <div
    class="full-columns"
    :class="containerClass">
    <div class="row">
      <div
        :class="{ 'd-none d-md-block': selectedId }"
        class="col">
        <div class="card">
          <div class="card-header">
            <div style="float:right;">
              <a
                href="javascript:void(0);"
                @click="loadList">
                  <icon :icon="['fas', 'sync-alt']" />
              </a>
            </div>
            {{ $t('trackings.trackings') }}
          </div>

          <div
            v-if="errors.length"
            class="container-error">
            <div
              v-for="(error, index) in errors"
              :key="index"
              class="alert custom-alert-danger">
              {{ $t(`errors.${error.where}`) }}
            </div>
          </div>

          <hero
            v-if="loading"
            icon="spinner"
            title="commons.loading"
            spin
          />

          <hero
            v-else-if="tasks.length === 0"
            icon="inbox"
            title="info.aboutTasks"
            desc="info.aboutTasksMore"
          />

          <ul
            v-else
            class="activity-list">
            <li
              :class="{ active: selectedId === task.execution.id }"
              v-for="task in tasks"
              :key="task.execution.id">
              <router-link
                :to="{
                  name: 'admin-tracking',
                  params: { id: task.execution.id },
                }"
                replace>
                <div
                  class="col-5 activity-name">
                  {{ task.execution.name }} — {{ task.node.name }}
                </div>
                <div class="col full-columns">
                  <div
                    v-for="(user, index) in task.notified_users"
                    :key="index">
                    {{ user.fullname }}
                  </div>
                </div>
                <div class="small"
                  :title="task.started_at | formatDate">
                    {{ task.started_at | relativeDate }}
                </div>
                <div class="activity-caret">
                  <icon :icon="['fas', 'caret-right']" />
                </div>
              </router-link>
            </li>
          </ul>
        </div>
      </div>

      <div v-if="selectedId" class="col-12 col-md-8">
        <admin-tracking
          :id="selectedId"
          :node="tasks[selectedId]"/>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment';
import { get } from '../utils/api';

export default {
  data() {
    return {
      tasks: [],
      timeline: [],
      loading: true,
      errors: [],
    };
  },
  mounted() {
    this.loadList();
  },
  methods: {
    loadList() {
      this.loading = true;
      this.errors = [];

      get('/log')
        .then((body) => {
          this.loading = false;

          const nodes = body.data;

          const arrToObj = arr =>
            arr.reduce((obj, item) => {
              // TODO: remove this
              // eslint-disable-next-line
              obj[item.execution.id] = item;
              return obj;
            }, {});

          this.tasks = arrToObj(nodes);
        })
        .catch((errors) => {
          this.loading = false;
          this.errors = errors;
        });
    },
  },
  filters: {
    relativeDate(val) {
      const date = new Date(val);
      const yesterday = new Date() - (24 * 60 * 60 * 1000);

      if (yesterday < date) {
        return moment(val).fromNow();
      }

      return moment(val).calendar();
    },
    formatDate(val) {
      return moment(val).format('LLLL');
    },
  },
  computed: {
    selectedId() {
      const { id } = this.$route.params;
      if (!id) {
        return null;
      }

      return id;
    },
    containerClass() {
      return {
        container: this.selectedId === null,
        'container-fluid': this.selectedId !== null,
      };
    },
  },
};
</script>

<style lang="scss" scoped>
.card, .row {
  flex: 1 1 auto;
}

.col {
  display: flex;
}

.container-error {
  padding: 30px;
  padding-bottom: 0;
}
</style>
