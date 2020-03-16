<template>
  <div :id="id">
  </div>
</template>

<script lang="ts">
import { HeatmapData, ColorRange, ValueRange, Margin, AxisLabels } from './types';

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

  @Prop({ required: false, default: DEFAULT_AXIS_PADDING })
  axisPadding!: number;

  @Prop({ required: false })
  axisLabels!: AxisLabels;

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
    return this.width - this.margin.right;
  }

  get boxHeight(): number {
    return this.height - this.margin.top;
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

    let axisX = d3.scaleBand()
      .range([0, this.boxWidth]);
    let axisY = d3.scaleBand()
       .range([this.boxHeight, 0]);

    this.svg.append('g')
      .attr('transform', `translate(0,${this.boxHeight})`)
      .call(d3.axisBottom(axisX))

    this.y = d3.scaleBand()
      .range([this.boxHeight, 0])
      .domain(this.axisY)
      .padding(this.axisPadding);

    this.svg.append('g')
      .call(d3.axisLeft(axisY));

    this.renderText();
  }

  renderText() {
    if(this.axisLabels === undefined) {
      return;
    }
    this.svg.selectAll('.y-label-text')
      .data([0])
      .enter()
      .append('text')
      .attr('x', -1 * this.margin.left)
      .attr('y', this.boxHeight / 2)
      .text(this.axisLabels.yLabel)
      .classed('value-text', true)
      .attr('font-family', 'Poppins, sans-serif')
      .attr('font-size', '10px')
      .style('font-weight', 'bold');

    this.svg.selectAll('.x-label-text')
      .data([0])
      .enter()
      .append('text')
      .attr('x', this.boxWidth / 2)
      .attr('y', this.boxHeight + this.margin.bottom)
      .text(this.axisLabels.xLabel)
      .classed('value-text', true)
      .attr('font-family', 'Poppins, sans-serif')
      .attr('font-size', '10px')
      .style('font-weight', 'bold');
    if(this.axisLabels.xRange !== undefined && this.axisLabels.xRange.length > 1) {
      this.svg.selectAll('.range-x-0')
        .data([0])
        .enter()
        .append('text')
        .attr('x', 0)
        .attr('y', this.boxHeight + this.margin.bottom)
        .text(this.axisLabels.xRange[0])
        .classed('value-text', true)
        .attr('font-family', 'Poppins, sans-serif')
        .attr('font-size', '10px')
        .style('font-weight', 'bold');
      this.svg.selectAll('.range-x-1')
        .data([0])
        .enter()
        .append('text')
        .attr('x', this.boxWidth - 20)
        .attr('y', this.boxHeight + this.margin.bottom)
        .text(this.axisLabels.xRange[1])
        .classed('value-text', true)
        .attr('font-family', 'Poppins, sans-serif')
        .attr('font-size', '10px')
        .style('font-weight', 'bold');
    }
    if(this.axisLabels.yRange !== undefined && this.axisLabels.yRange.length > 1) {
      this.svg.selectAll('.range-y-1')
        .data([0])
        .enter()
        .append('text')
        .attr('x', -15)
        .attr('y', 10)
        .text(this.axisLabels.yRange[1])
        .classed('value-text', true)
        .attr('font-family', 'Poppins, sans-serif')
        .attr('font-size', '10px')
        .style('font-weight', 'bold');
      this.svg.selectAll('.range-y-0')
        .data([0])
        .enter()
        .append('text')
        .attr('x', -15)
        .attr('y', this.boxHeight - 10)
        .text(this.axisLabels.yRange[0])
        .classed('value-text', true)
        .attr('font-family', 'Poppins, sans-serif')
        .attr('font-size', '10px')
        .style('font-weight', 'bold');
    }
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
      .style('padding', '5px');
  }

  mouseOver(d: any, i: number, node: any): void {
    this.$emit('tooltip', {
      displayed: true
    });
  }

  mouseMove(d: any, i: number, node: any): void {
    this.$emit('tooltip', {
      displayed: true,
      x: d3.event.clientX - 125,
      y: d3.event.clientY - 60,
      content: `value: ${d.value}`
    });
  }
  mouseLeave(d: any, i: number, node: any): void {
    this.$emit('tooltip', {
      displayed: false
    });
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
        .on('mouseover', this.mouseOver)
        .on('mousemove', this.mouseMove)
        .on('mouseleave', this.mouseLeave)
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
