<template>
  <div :id="id">
  </div>
</template>

<script lang="ts">
import { HeatmapData, ColorRange, ValueRange, Margin } from './types';

import * as d3 from 'd3';

import { Component, Vue, Prop, Watch } from 'vue-property-decorator';

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
    top: 0,
    right: 0,
    bottom: 0,
    left: 0
  }
}

const DEFAULT_AXIS_PADDING = 0.01;

@Component
export default class Heatmap extends Vue {
  @Prop({ required: true })
  id!: string;

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

  //TODO: use SVG type
  svg: any = null;
  tooltip: any = undefined;
  x: any = undefined;
  y: any = undefined;

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

  @Watch('data')
  onDataChange() {
    this.renderHeatmap();
  }

  renderScaleBand() {
    this.x = d3.scaleBand()
      .range([0, this.boxWidth])
      .domain(this.axisX)
      .padding(this.axisPadding);

    this.svg.append('g')
      .attr('transform', `translate(0,${this.boxHeight})`)
      .call(d3.axisBottom(this.x))

    this.y = d3.scaleBand()
      .range([this.boxHeight, 0])
      .domain(this.axisY)
      .padding(this.axisPadding);

    this.svg.append('g')
      .call(d3.axisLeft(this.y));
  }

    renderTooltip() {
    this.tooltip = d3.select(`#${this.id}`)
      .append('div')
      .style('opacity', 0)
      .attr('class', 'tooltip')
      .style('background-color', 'white')
      .style('border', 'solid')
      .style('border-width', '2px')
      .style('border-radius', '5px')
      .style('padding', '5px')
  }

  mouseOver(d: HeatmapData, i: number, node: any) {
    this.tooltip
      .style('opacity', 1)
    d3.select(node[i])
      .style('stroke', 'black')
      .style('opacity', 1)
  }

  mouseMove(d: HeatmapData, i: number, node: any) {
    this.tooltip
      .html('value: ' + d.value)
      .style('left', (d3.mouse(node[i])[0]+10) + 'px')
      .style('top', (d3.mouse(node[i])[1]) + 'px')
  }

  mouseLeave(d: HeatmapData, i: number, node: any) {
    this.tooltip
      .style('opacity', 0)
    d3.select(node[i])
      .style('stroke', 'none')
      .style('opacity', 0.8)
  }

  renderHeatmap() {
    this.svg.selectAll().remove();
    this.renderScaleBand();

    let myColor = d3.scaleLinear()
      .range(this.colorRangeList)
      .domain(this.valueRangeList)

    let heatmap = this.svg.selectAll()
      .data(this.data)
      .enter()
        .append('rect')
          .attr('x', (d: HeatmapData) => { return this.x(d.x) } )
          .attr('y', (d: HeatmapData) => {  return this.y(d.y) } )
          .attr('width', this.x.bandwidth() )
          .attr('height', this.y.bandwidth() )
          .style('fill', (d: HeatmapData) => { return myColor(d.value) } )
        .on('mouseover', this.mouseOver )
        .on('mousemove', this.mouseMove )
        .on('mouseleave', this.mouseLeave )
  }

  mounted() {
    this.svg = d3.select(`#${this.id}`)
      .append('svg')
        .attr('width', this.boxWidth + this.margin.left + this.margin.right)
        .attr('height', this.boxHeight + this.margin.top + this.margin.bottom)
      .append('g')
        .attr('transform',
              `translate(${this.margin.left}, ${this.margin.top})`);

    this.renderTooltip();
    this.renderHeatmap();
  }
}
</script>
