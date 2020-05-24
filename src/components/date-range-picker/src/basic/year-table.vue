<template>
  <table @click="handleYearTableClick" @mousemove="handleMouseMove" class="el-year-table">
    <tbody>

    <tr v-for="(row, key) in rows" :key="key">
      <td :class="getCellStyle(cell)" v-for="(cell, key) in row" :key="key">
        <div>
          <a class="cell">{{ cell.text }}</a>
        </div>
      </td>
    </tr>

    </tbody>
  </table>
</template>

<script type="text/babel">
  import { hasClass } from 'element-ui/src/utils/dom';
  import { isDate, range, nextDate, getDayCountOfYear } from 'element-ui/src/utils/date-util';
  import { arrayFindIndex, coerceTruthyValueToArray } from 'element-ui/src/utils/util';

  const datesInYear = year => {
    const numOfDays = getDayCountOfYear(year);
    const firstDay = new Date(year, 0, 1);
    return range(numOfDays).map(n => nextDate(firstDay, n));
  };

  export default {
    props: {
      value: {},
      defaultValue: {
        validator(val) {
          // null or valid Date Object
          return val === null || (val instanceof Date && isDate(val));
        }
      },
      date: {},
      rangeState: {
        default() {
          return {
            endYear: null,
            selecting: false
          };
        }
      },
      startYear: Number,
      minYear: Number,
      maxYear: Number,
    },

    data() {
      return {
        tableRows: [ [], [], [] ],
        lastYear: 0
      }
    },

    watch: {
      'rangeState.endYear'(newVal) {
        this.markRange(this.minYear, newVal);
      },

      minYear(newVal, oldVal) {
        if (newVal !== oldVal) {
          this.markRange(this.minYear, this.maxYear);
        }
      },

      maxYear(newVal, oldVal) {
        if (newVal !== oldVal) {
          this.markRange(this.minYear, this.maxYear);
        }
      }
    },

    computed: {
      rows() {
        const rows = this.tableRows;
        const selectedDate = [];
        const now = Math.floor((new Date()).getFullYear() / 10) * 10;

        for (let i = 0; i < 3; i++) {
          const row = rows[i];
          for (let j = 0; j < 4; j++) {
            let cell = row[j];
            if (!cell) {
              cell = { row: i, column: j, type: 'normal', inRange: false, start: false, end: false };
            }

            cell.type = 'normal';

            const index = i * 4 + j;
            const time = this.startYear + index;
            cell.inRange = time >= this.minYear && time <= this.maxYear;
            cell.start = this.minYear && time === this.minYear;
            cell.end = this.maxYear && time === this.maxYear;
            const isToday = time === now;

            if (isToday) {
              cell.type = 'today';
            }
            cell.text = time;
            let cellDate = new Date(time);

            this.$set(row, j, cell);
          }
        }
        return rows;
      }
    },

    methods: {
      getCellStyle(cell) {
        const style = {};
        const today = new Date();

        style.today = today.getFullYear() === cell.text;

        if (cell.inRange) {
          style['in-range'] = true;

          if (cell.start) {
            style['start-date'] = true;
          }

          if (cell.end) {
            style['end-date'] = true;
          }
        }
        return style;
      },

      handleYearTableClick(event) {
        const target = event.target;
        if (target.tagName === 'A') {
          const year = Number(target.textContent || target.innerText);

          if (!this.rangeState.selecting) {
            this.$emit('pick', {minYear: year, maxYear: null});
            this.rangeState.selecting = true;
          } else {
            if (year >= this.minYear) {
              this.$emit('pick', {minYear: this.minYear, maxYear: year});
            } else {
              this.$emit('pick', {minYear: year, maxYear: this.minYear});
            }
            this.rangeState.selecting = false;
          }
        }
      },

      handleMouseMove(event) {
        if (!this.rangeState.selecting) return;

        let target = event.target;
        if (target.tagName === 'DIV') {
          target = target.childNodes[0];
        }
        if (target.tagName === 'TD') {
          let temp = target.childNodes[0];
          if (temp) {
            target = temp.childNodes[0]
          }
        }
        if (target.tagName !== 'A') return;

        const year = target.textContent || target.innerText;
        if (year !== this.lastYear) {
          this.year = year
          this.$emit('changerange', {
            minYear: this.minYear,
            maxYear: this.maxYear,
            rangeState: {
              selecting: true,
              endYear: year
            }
          });
        }
      },

      markRange(minYear, maxYear) {
        minYear = Number(minYear);
        maxYear = maxYear ? Number(maxYear) : Number(minYear);
        [minYear, maxYear] = [Math.min(minYear, maxYear), Math.max(minYear, maxYear)];

        const rows = this.rows;
        for (let i = 0, k = rows.length; i < k; i++) {
          const row = rows[i];
          for (let j = 0, l = row.length; j < l; j++) {

            const cell = row[j];
            const time = cell.text

            cell.inRange = minYear && time >= minYear && time <= maxYear;
            cell.start = minYear && time === minYear;
            cell.end = maxYear && time === maxYear;
          }
        }
      },

    }
  };
</script>

<style scoped>
.el-year-table {
  table-layout: fixed;
  width: 100%;
  margin: -1px;
  font-size: 12px;
  border-collapse: collapse;
}
.el-year-table td.today .cell {
  color: #409EFF;
  font-weight: 700;
}
.el-year-table td .cell {
  width: 60px;
  height: 36px;
  display: block;
  line-height: 36px;
  color: #606266;
  margin: 0 auto;
  border-radius: 18px;
}
.el-year-table td {
  text-align: center;
  cursor: pointer;
  padding: 8px 0;
}
.el-year-table td.start-date div {
  border-top-left-radius: 24px;
  border-bottom-left-radius: 24px;
  color: #fff;
}
.el-year-table td.end-date div {
  border-top-right-radius: 24px;
  border-bottom-right-radius: 24px;
  color: #fff;
}
.el-year-table td div {
  height: 48px;
  padding: 6px 0;
  box-sizing: border-box;
}
.el-year-table td.in-range div, .el-year-table td.in-range div:hover {
  background-color: #F2F6FC;
}
.el-year-table td.end-date .cell, .el-year-table td.start-date .cell {
    color: #fff;
    background-color: #409EFF;
}
</style>
