<template>
  <transition name="el-zoom-in-top" @after-leave="$emit('dodestroy')">
    <div
      v-show="visible"
      class="el-picker-panel el-date-range-picker el-popper"
      :class="[{
        'has-sidebar': $slots.sidebar || shortcuts
      }, popperClass]">
      <div class="el-picker-panel__body-wrapper">
        <slot name="sidebar" class="el-picker-panel__sidebar"></slot>
        <div class="el-picker-panel__sidebar" v-if="shortcuts">
          <button
            type="button"
            class="el-picker-panel__shortcut"
            v-for="(shortcut, key) in shortcuts"
            :key="key"
            @click="handleShortcutClick(shortcut)">{{shortcut.text}}</button>
        </div>
        <div class="el-picker-panel__body">
          <div class="el-picker-panel__content el-date-range-picker__content is-left">
            <div class="el-date-range-picker__header">
              <button
                type="button"
                @click="leftPrevYear"
                class="el-picker-panel__icon-btn el-icon-d-arrow-left"></button>
              <button
                type="button"
                v-if="unlinkPanels"
                @click="leftNextYear"
                :disabled="!enableYearArrow"
                :class="{ 'is-disabled': !enableYearArrow }"
                class="el-picker-panel__icon-btn el-icon-d-arrow-right"></button>
              <div>{{ leftLabel }}</div>
            </div>
            <year-table
              :start-year="leftYear"
              :min-year="minYear"
              :max-year="maxYear"
              selection-mode="range"
              :date="leftDate"
              :default-value="defaultValue"
              :min-date="minDate"
              :max-date="maxDate"
              :range-state="rangeState"
              :disabled-date="disabledDate"
              @changerange="handleChangeRange"
              @pick="handleRangePick">
            </year-table>
          </div>
          <div class="el-picker-panel__content el-date-range-picker__content is-right">
            <div class="el-date-range-picker__header">
              <button
                type="button"
                v-if="unlinkPanels"
                @click="rightPrevYear"
                :disabled="!enableYearArrow"
                :class="{ 'is-disabled': !enableYearArrow }"
                class="el-picker-panel__icon-btn el-icon-d-arrow-left"></button>
              <button
                type="button"
                @click="rightNextYear"
                class="el-picker-panel__icon-btn el-icon-d-arrow-right"></button>
              <div>{{ rightLabel }}</div>
            </div>
            <year-table
              :start-year="rightYear"
              :min-year="minYear"
              :max-year="maxYear"
              selection-mode="range"
              :date="rightDate"
              :default-value="defaultValue"
              :min-date="minDate"
              :max-date="maxDate"
              :range-state="rangeState"
              :disabled-date="disabledDate"
              @changerange="handleChangeRange"
              @pick="handleRangePick">
            </year-table>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/babel">
  import {
    isDate,
    modifyWithTimeString,
    prevYear,
    nextYear,
    nextMonth
  } from 'element-ui/src/utils/date-util';
  import Clickoutside from 'element-ui/src/utils/clickoutside';
  import Locale from 'element-ui/src/mixins/locale';
  import MonthTable from '../basic/month-table';
  import YearTable from '../basic/year-table';
  import ElInput from 'element-ui/packages/input';
  import ElButton from 'element-ui/packages/button';

  const calcDefaultValue = (defaultValue) => {
    if (Array.isArray(defaultValue)) {
      return [new Date(defaultValue[0]), new Date(defaultValue[1])];
    } else if (defaultValue) {
      return [new Date(defaultValue), nextMonth(new Date(defaultValue))];
    } else {
      return [new Date(), nextMonth(new Date())];
    }
  };
  export default {
    mixins: [Locale],

    directives: { Clickoutside },

    created() {
      let date = new Date()
      this.rightYear = Math.floor(date.getFullYear() / 10) * 10
      this.leftYear = this.rightYear - 12
    },

    computed: {
      btnDisabled() {
        return !(this.minDate && this.maxDate && !this.selecting && this.isValidValue([this.minDate, this.maxDate]));
      },

      leftLabel() {
        const yearTranslation = '年';
        return this.leftYear + ' ' + yearTranslation + ' - ' + (this.leftYear + 11) + ' ' + yearTranslation;
      },

      rightLabel() {
        const yearTranslation = '年';
        return this.rightYear + ' ' + yearTranslation + ' - ' + (this.rightYear + 11) + ' ' + yearTranslation;
      },

      /*leftYear() {
        return this.leftDate.getFullYear();
      },*/

      /*rightYear() {
        return this.rightDate.getFullYear() === this.leftDate.getFullYear() ? this.leftDate.getFullYear() + 1 : this.rightDate.getFullYear();
      },*/

      enableYearArrow() {
        return this.unlinkPanels && this.rightYear > this.leftYear + 1;
      }
    },

    data() {
      return {
        popperClass: '',
        value: [],
        defaultValue: null,
        defaultTime: null,
        minDate: '',
        maxDate: '',
        leftDate: new Date(),
        rightDate: nextYear(new Date()),
        rangeState: {
          endDate: null,
          selecting: false,
          row: null,
          column: null
        },
        shortcuts: '',
        visible: '',
        disabledDate: '',
        format: '',
        arrowControl: false,
        unlinkPanels: false,

        leftYear: '',
        rightYear: '',
        minYear: 0,
        maxYear: 0
      };
    },

    watch: {
      value(newVal) {
        if (!newVal) {
          this.minDate = null;
          this.maxDate = null;
        } else if (Array.isArray(newVal)) {
          this.minDate = isDate(newVal[0]) ? new Date(newVal[0]) : null;
          this.maxDate = isDate(newVal[1]) ? new Date(newVal[1]) : null;
          if (this.minDate) {
            this.leftDate = this.minDate;
            if (this.unlinkPanels && this.maxDate) {
              const minDateYear = this.minDate.getFullYear();
              const maxDateYear = this.maxDate.getFullYear();
              this.rightDate = minDateYear === maxDateYear
                ? nextYear(this.maxDate)
                : this.maxDate;
            } else {
              this.rightDate = nextYear(this.leftDate);
            }
          } else {
            this.leftDate = calcDefaultValue(this.defaultValue)[0];
            this.rightDate = nextYear(this.leftDate);
          }
        }
      },

      defaultValue(val) {
        if (!Array.isArray(this.value)) {
          const [left, right] = calcDefaultValue(val);
          this.leftDate = left;
          this.rightDate = val && val[1] && left.getFullYear() !== right.getFullYear() && this.unlinkPanels
            ? right
            : nextYear(this.leftDate);
        }
      }
    },

    methods: {
      handleClear() {
        this.minYear = null;
        this.maxYear = null;
        let date = new Date()
        this.rightYear = Math.floor(date.getFullYear() / 10) * 10
        this.leftYear = this.rightYear - 12
        this.$emit('pick', null);
      },

      handleChangeRange(val) {
        this.minYear = val.minYear;
        this.maxYear = val.maxYear;
        this.rangeState = val.rangeState;
      },

      handleRangePick(val, close = true) {
        const defaultTime = this.defaultTime || [];
        const minYear = val.minYear
        const maxYear = val.maxYear
        if (this.maxYear === maxYear && this.minYear === minYear) {
          return;
        }
        this.onPick && this.onPick(val);
        this.maxYear = maxYear;
        this.minYear = minYear;

        // workaround for https://github.com/ElemeFE/element/issues/7539, should remove this block when we don't have to care about Chromium 55 - 57
        setTimeout(() => {
          this.maxYear = maxYear;
          this.minYear = minYear;
        }, 10);
        if (!close) return;
        this.handleConfirm();
      },

      handleShortcutClick(shortcut) {
        if (shortcut.onClick) {
          shortcut.onClick(this);
        }
      },

      // leftPrev*, rightNext* need to take care of `unlinkPanels`
      leftPrevYear() {
        this.leftYear = this.leftYear - 12
        if (!this.unlinkPanels) {
          this.rightYear = this.rightYear - 12
        }
      },

      rightNextYear() {
        if (!this.unlinkPanels) {
          this.leftYear = this.leftYear + 12
        }
        this.rightYear = this.rightYear + 12
      },

      // leftNext*, rightPrev* are called when `unlinkPanels` is true
      leftNextYear() {
        this.leftDate = nextYear(this.leftDate);
      },

      rightPrevYear() {
        this.rightDate = prevYear(this.rightDate);
      },

      handleConfirm(visible = false) {
        if (this.isValidValue([this.minYear, this.maxYear])) {
          this.$emit('pick', [this.minYear, this.maxYear], visible, true);
        }
      },

      isValidValue(value) {
        return Array.isArray(value) &&
          value && value[0] && value[1] &&
          (
          typeof this.disabledDate === 'function'
            ? !this.disabledDate(value[0]) && !this.disabledDate(value[1])
            : true
        );
      },

      resetView() {
        // NOTE: this is a hack to reset {min, max}Date on picker open.
        // TODO: correct way of doing so is to refactor {min, max}Date to be dependent on value and internal selection state
        //       an alternative would be resetView whenever picker becomes visible, should also investigate date-panel's resetView
        this.minDate = this.value && isDate(this.value[0]) ? new Date(this.value[0]) : null;
        this.maxDate = this.value && isDate(this.value[0]) ? new Date(this.value[1]) : null;
      }
    },

    components: { MonthTable, YearTable, ElInput, ElButton }
  };
</script>
