<template>
  <div class="form-group position-relative">
    <label v-uni-for="name">{{label}}</label>
    <date-picker
      v-model="date"
      :config="config"
      :disabled="disabled"
      :placeholder="placeholder"
      :data-test="dataTest"
    />
    <div v-if="(validator && validator.errorCount) || error" class="invalid-feedback d-block">
      <div v-for="(error, index) in validator.errors.get(this.name)" :key="index">{{error}}</div>
      <div v-if="error">{{error}}</div>
    </div>
    <small v-if="helper" class="form-text text-muted">{{helper}}</small>
  </div>
</template>

<script>
import {createUniqIdsMixin} from 'vue-uniq-ids';
import ValidationMixin from './mixins/validation';
import DataFormatMixin from "./mixins/DataFormat";
import datePicker from 'vue-bootstrap-datetimepicker';
import 'pc-bootstrap4-datetimepicker/build/css/bootstrap-datetimepicker.css';
import moment from 'moment-timezone';
import {getLang, getTimezone, getUserDateFormat, getUserDateTimeFormat} from '../dateUtils';

const uniqIdsMixin = createUniqIdsMixin();

moment.tz.setDefault(getTimezone());

export default {
  mixins: [uniqIdsMixin, ValidationMixin, DataFormatMixin],
  components: {
    datePicker
  },
  props: {
    name: String,
    placeholder: String,
    label: String,
    error: String,
    helper: String,
    dataFormat: String,
    value: String,
    inputClass: {type: [String, Array, Object], default: 'form-control'},
    dataTest: String,
    disabled: null,
  },
  data() {
    return {
      date: null,
      config: {
        format: getUserDateFormat(),
        timeZone: getTimezone(),
        locale: getLang(),
        useCurrent: false,
        showClose: true,
        icons: {
          time: 'far fa-clock',
          date: 'far fa-calendar',
          up: 'fas fa-arrow-up',
          down: 'fas fa-arrow-down',
          previous: 'fas fa-chevron-left',
          next: 'fas fa-chevron-right',
          today: 'fas fa-calendar-check',
          clear: 'far fa-trash-alt',
          close: 'far fa-times-circle'
        }
      },
    }
  },
  watch: {
    date: {
      deep: true,
      handler(date) {
        if (!this.date) {
          return;
        }

        this.$emit('input', this.generateDate(moment(date, this.config.format)).toISOString());
      },
    },
    value(value) {
      const date = this.generateDate(value);
      if (date.toISOString() !== moment(this.date, this.config.format).toISOString()) {
        this.date = date;
      }
    },
    dataFormat: {
      immediate: true,
      handler() {
        this.config.format = this.dataFormat === 'datetime'
          ? getUserDateTimeFormat()
          : getUserDateFormat();

        this.date = this.generateDate();
      }
    },
  },
  methods: {
    generateDate(value = this.value) {
      let date = moment(value);

      if (!date.isValid()) {
        date = moment();
      }

      if (this.dataFormat !== 'datetime') {
        date.startOf('day');
      }

      return date;
    },
  },
};
</script>

<style>
  .inspector-container .bootstrap-datetimepicker-widget.dropdown-menu {
    font-size: 11px;
  }
</style>
