<template>
  <div id="heatmap">
  </div>
</template>

<script lang="ts">
import { HeatmapData, ColorRange, ValueRange, Margin } from './types';

import * as d3 from 'd3';

import { Component, Vue, Prop } from 'vue-property-decorator';

const DEFAULT_COLOR_RANGE: ColorRange = {
  min: 'white',
  max: '#69b3a2'
}

const DEFAULT_VALUE_RANGE: ValueRange = {
  min: 1,
  max: 100
}

const DEFAULT_STYLES = {
  width: 450,
  height: 450,
  margin: {
    top: 40,
    right: 40,
    bottom: 40,
    left: 40
  }
}

const DEFAULT_AXIS_PADDING = 0.01;

@Component
export default class Heatmap extends Vue {
  @Prop({ required: true })
  data!: [HeatmapData];

  @Prop({ required: true })
  axisX!: [string]

  @Prop({ required: true })
  axisY!: [string]

  @Prop({ required: false, default: DEFAULT_STYLES.width })
  width!: number;

  @Prop({ required: false, default: DEFAULT_STYLES.height })
  height!: number;

  @Prop({ required: false, default: () => DEFAULT_STYLES.margin })
  margin!: Margin;

  @Prop({ required: false, default: () => DEFAULT_COLOR_RANGE })
  colorRange!: ColorRange;

  @Prop({ required: false, default: () => DEFAULT_VALUE_RANGE })
  valueRange!: ValueRange;

  @Prop({required: false, default: DEFAULT_AXIS_PADDING })
  axisPadding!: number;

  get valueRangeList(): [number, number] {
    return [this.valueRange.min, this.valueRange.max]
  }

  get colorRangeList(): any {
    //TODO: change return type
    return [this.colorRange.min, this.colorRange.max]
  }

  get boxWidth(): number {
    return this.width - this.margin.left - this.margin.right;
  }

  get boxHeight(): number {
    return this.height - this.margin.top - this.margin.bottom;
  }

  mounted() {
    let svg = d3.select('#heatmap')
      .append('svg')
        .attr('width', this.boxWidth + this.margin.left + this.margin.right)
        .attr('height', this.boxHeight + this.margin.top + this.margin.bottom)
      .append('g')
        .attr('transform',
              `translate(${this.margin.left}, ${this.margin.top})`);

    let x = d3.scaleBand()
      .range([0, this.boxWidth])
      .domain(this.axisX)
      .padding(this.axisPadding);

    svg.append('g')
      .attr('transform', `translate(0,${this.boxHeight})`)
      .call(d3.axisBottom(x))

    let y = d3.scaleBand()
      .range([this.boxHeight, 0])
      .domain(this.axisY)
      .padding(this.axisPadding);
    svg.append('g')
      .call(d3.axisLeft(y));

    let myColor = d3.scaleLinear()
      .range(this.colorRangeList)
      .domain(this.valueRangeList)

    let heatmap = svg.selectAll()
      .data(this.data)
      .enter()

    //TODO: remove as any
    for(let i = 0; i < this.data.length; i++) {
      heatmap
        .append('rect')
        .attr('x', x(this.data[i].x) as any )
        .attr('y', y(this.data[i].y) as any )
        .attr('width', x.bandwidth() )
        .attr('height', y.bandwidth() )
        .style('fill', myColor(this.data[i].value) )
    }
  }
}
</script>
