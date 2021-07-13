<template>
  <div :id="id">
  </div>
</template>

<script lang="ts">
import { HeatmapData, ValueRange, Margin, AxisLabels } from './types';

import * as d3 from 'd3';

import { Component, Vue, Prop, Watch } from 'vue-property-decorator';

const DEFAULT_COLOR_RANGE = ['white', '#69b3a2'];

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

  @Prop({ required: false })
  width!: number;

  @Prop({ required: false })
  height!: number;

  @Prop({ required: false, default: () => DEFAULT_STYLES.margin })
  margin!: Margin;

  @Prop({ required: false, default: () => DEFAULT_COLOR_RANGE })
  colorRange!: any[];

  @Prop({ required: false, default: () => DEFAULT_VALUE_RANGE })
  valueRange!: ValueRange;

  @Prop({ required: false, default: DEFAULT_AXIS_PADDING })
  axisPadding!: number;

  @Prop({ required: false })
  axisLabels!: AxisLabels;

  //TODO: use SVG type
  d3Node: any;
  svg: any = null;
  tooltip: any = undefined;
  x: any = undefined;
  y: any = undefined;

  get valueRangeList(): number[] {
    let range = [];
    const valueDiff = this.valueRange.max - this.valueRange.min;
    const tickValue = valueDiff / this.colorRange.length;
    for(let i = 1; i <= this.colorRange.length; i++) {
      range.push(tickValue * i);
    }
    return range;
  }

  get minOfClientHeightAndWidth(): number {
    return Math.min(this.$el.clientWidth, this.$el.clientHeight);
  }

  get boxWidth(): number {
    const width = this.width || this.minOfClientHeightAndWidth;
    return width - this.margin.right;
  }

  get boxHeight(): number {
    const height = this.height || this.minOfClientHeightAndWidth;
    return height - this.margin.top;
  }

  @Watch('data')
  onDataChange() {
    this.createSvg();
    this.renderHeatmap();
  }

  createSvg() {
    this.d3Node.selectAll('svg *').remove();

    this.svg = this.d3Node
      .select('svg')
        .attr('width', this.boxWidth + this.margin.left + this.margin.right)
        .attr('height', this.boxHeight + this.margin.top + this.margin.bottom)
      .append('g')
        .attr('transform',
              `translate(${this.margin.left}, ${this.margin.top})`);
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
      .attr('x', this.boxWidth / 2 - 10)
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
        .attr('x', 10)
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
      x: d3.event.clientX - 40,
      y: d3.event.clientY - 60,
      content: d.value,
      xValue: d.x,
      yValue: d.y,
      id: this.id
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
      .domain(this.valueRangeList)
      .range(this.colorRange);

    let heatmap = this.svg.selectAll()
      .data(this.data)
      .enter()
        .append('rect')
          .attr('x', (d: HeatmapData) => { return this.x(d.x) } )
          .attr('y', (d: HeatmapData) => { return this.y(d.y) + 1} )
          .attr('width', this.x.bandwidth() )
          .attr('height', this.y.bandwidth() )
          .style(
            'fill', 
            (d: HeatmapData) => { 
              if(d.value === null || d.value === undefined) { return 'none'; }
              return myColor(d.value);
            }
          )
        .on('mouseover', this.mouseOver)
        .on('mousemove', this.mouseMove)
        .on('mouseleave', this.mouseLeave)
  }

  mounted() {
    this.d3Node = d3.select(`#${this.id}`)
    this.d3Node.append('svg');
    this.createSvg();

    this.renderTooltip();
    this.renderHeatmap();
  }
}
</script>
