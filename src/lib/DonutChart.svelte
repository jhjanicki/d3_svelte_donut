<script>
  import DonutArc from "./DonutArc.svelte";
  import HiddenArc from "./HiddenArc.svelte";
  import DonutLabel from "./DonutLabel.svelte";
  import Legend from "./Legend.svelte";
  import data from "../data/data.json";
  import * as d3 from "d3";

  const categoriesSorted = [
    "Critically Endangered",
    "Endangered",
    "Vulnerable",
    "Near Threatened",
    "Least Concern",
    "Data Deficient",
  ];
  const colorScale = d3
    .scaleOrdinal()
    .domain(categoriesSorted)
    .range(["#b30000", "#e34a33", "#fc8d59", "#fdbb84", "#fee8c8", "#d9d9d9"]);
  
    // all variables related to dimensions
  const length = 400;
  const marginLength = 30;
  const categoryHeight = 20;
  const margin = {
      "top": marginLength,
      "left": marginLength,
      "bottom": marginLength,
      "right": marginLength
  }
  //prep data
  const sumSortData = (data) => {
      //summarise data & calculate ratio & raw count for each category
      const rolledData = data.map((entry) =>
          d3.map(
              d3.rollup(data, v => v.length, d => d.redlistCategory),
              ([category, result]) => ({
                  category: category,
                  ratio: (result / data.length) * 100,
                  quantity: result,
              })
          )
      )[0]
      // sort the data based on the order of categories
      const order = {}; // map for efficient lookup of sortIndex
      for (let i = 0; i < categoriesSorted.length; i++) {
          order[categoriesSorted[i]] = i;
      }
      return rolledData.sort((a, b) => (order[a.category] - order[b.category]));
  }

  const createPieData = (data) => {
      //add sort null if want to order segments by category, remove if want to order by ratio 
      const pie = d3.pie().value((d) => d.ratio).sort(null);
      //pie(data) creates startAngle, endAngle for each data point
      const arcGenerator = d3
          .arc()
          .outerRadius(length / 2)
          .innerRadius(length / 3);

      return pie(data).map((d) => {
          return {
              data: d.data,
              path: arcGenerator.startAngle(d.startAngle).endAngle(d.endAngle)(),
              fill: colorScale(d.data.category),
              centroid: arcGenerator.centroid(d),
          };
      });
  }
  //final dataset
  const dataSummed = sumSortData(data);
  const pieData = createPieData(dataSummed);
</script>

<svg
  width={length + margin.left + margin.right}
  height={length + margin.top + margin.bottom}
>
  <g transform="translate({margin.left + length / 2},{margin.top + length / 2})">
    <DonutArc
      data={pieData}
      {colorScale}
    />
    <HiddenArc data={pieData}/>
    <DonutLabel
      data={pieData}
    />
  </g>
  <g transform="translate({(length)/2-40},{length/2-20})"> <!-- lazy to calculate so the centering is manual  -->
    <Legend
      data={dataSummed}
      {categoryHeight}
      {colorScale}
    ></Legend>
  </g>
</svg>