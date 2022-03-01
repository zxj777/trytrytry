<template>
  <div ref="ganttContainer">
    <svg xmlns="http://www.w3.org/2000/svg">
      <g class="gantt-layout">
        <rect
          x="0"
          y="0"
          :width="backgroundConfig.width"
          :height="backgroundConfig.height"
        ></rect>
      </g>
      <g class="gantt-date"></g>
      <g class=""></g>
      <g></g>
      <g></g>
    </svg>
  </div>
</template>

<script>
/* eslint-disable @typescript-eslint/explicit-module-boundary-types */
import dayjs from "dayjs";
export default {
  props: {
    taskParams: {
      type: Array,
      default() {
        return [];
      },
    },
    optionParams: {
      type: Object,
      default() {
        return {};
      },
    },
  },
  data() {
    return {
      taskList: [],
      options: {},
      drawConfig: {
        ganttStart: 0,
        ganttEnd: 0,
      },
      backgroundConfig: {
        width: 0,
        height: 0,
      },
      dates: [],
      drawDates: [],
    };
  },
  created() {
    this.mergeOptions(this.optionParams);
    this.processTaskParams(this.taskParams);
    this.setDates();
    this.getDrawDates();
    this.handleLayout();
  },
  methods: {
    mergeOptions(optionParams) {
      const defaultOptions = {
        columnWidth: 30,
        headerHeight: 50,
        padding: 20,
        barHeight: 24,
      };
      this.options = Object.assign({}, defaultOptions, optionParams);
    },
    processTaskParams(taskParams) {
      this.taskList = taskParams.map((task) => {
        const _startTime = dayjs(task.startTime);
        const _endTime = dayjs(task.endTime);
        return {
          ...task,
          _startTime,
          _endTime,
        };
      });
    },
    setDates() {
      let ganttStart, ganttEnd;
      // 设置任务起点终点
      for (let task of this.taskList) {
        if (!ganttStart || task._start < ganttStart) {
          ganttStart = task._start;
        }
        if (!ganttEnd || task._end > ganttEnd) {
          ganttEnd = task._end;
        }
      }

      // 扩大显示，视图默认day模式
      ganttStart = dayjs(ganttStart).minute(1, "month");
      ganttEnd = dayjs(ganttEnd).add(1, "month");

      this.drawConfig = {
        ganttStart,
        ganttEnd,
      };

      const dates = [];
      let curDate = dayjs(ganttStart);
      dates.push(curDate);

      while (curDate && curDate < ganttEnd) {
        curDate = dayjs(curDate).add(1, "day");
        dates.push(curDate);
      }
      this.dates = dates;
    },
    handleLayout() {
      this.setBackground();
    },
    setBackground() {
      const { headerHeight, padding, barHeight, columnWidth } = this.options;
      const width = this.dates.length * columnWidth;
      const height =
        headerHeight + padding + (barHeight + padding) * this.taskList.length;
      this.backgroundConfig = {
        width,
        height,
      };
    },
    // getDrawDates() {},
  },
};
</script>
