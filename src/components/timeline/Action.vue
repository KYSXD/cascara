<template>
  <div :id="action.id" class="timeline-action" :class="{'highlight': highlight}">
    <span class="timeline-dot" />

    <div class="card">
      <div class="card-header">
        <div>
          <div class="actions">
            <icon
              class="toggle"
              @click="toggleCollapse"
              :icon="collapseClassName"
            />
          </div>
          {{ action.node.name }}
          &bull;
          <small>{{ action.finished_at | relativeDate }}</small>
        </div>
      </div>
      <div class="card-body" v-if="!collapse">
        <div
          v-for="(actor, identifier) in action.actors.items"
          :key="identifier">
          <div v-if="actor">
            <p>
              <b>{{ actor.user.fullname }}</b>
              llenó la siguiente información
            </p>
            <table class="table table-sm table-bordered">
              <tbody
                v-for="(form, key) in actor.forms"
                :key="key">
                <tr class="form-group">
                  <td
                    :title="`#${form.ref}`"
                    :rowspan="form.inputs.item_order.length + 1">
                  </td>
                </tr>
                <tr
                  v-for="input in listInputs(form.inputs)"
                  :key="input.name">
                  <td scope="row">{{ input.label }}</td>
                  <td><value-render :input="input" /></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment';

export default {
  props: ['action', 'highlight'],
  data() {
    return {
      collapse: false,
    };
  },
  methods: {
    toggleCollapse() {
      this.collapse = !this.collapse;
    },
    listInputs(inputs) {
      return inputs.item_order
        .map(key => inputs.items[key])
        .filter(input => !input.hidden);
    },
  },
  computed: {
    collapseClassName() {
      const response = ['fas'];
      if (this.collapse) {
        response.push('chevron-down');
      } else {
        response.push('chevron-up');
      }

      return response;
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
  },
};
</script>

<style lang="scss" scoped>
table {
  td {
    word-break: break-all;
  }
}

.actions .toggle {
  color: #888;
  cursor: pointer;
}

.actions .toggle:hover {
  color: #333;
}
</style>
