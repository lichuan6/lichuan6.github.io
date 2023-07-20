# Render all chartjs charts in react typescript tailwindcss projects

In this tutorial, we'll show you how to render all chartjs charts in React project.

# Table of Contents

<!--ts-->

- [Render all chartjs charts in react typescript tailwindcss projects](#render-all-chartjs-charts-in-react-typescript-tailwindcss-projects)
- [Table of Contents](#table-of-contents)
- [Demo(seeing is believing)](#demoseeing-is-believing)
- [Start NextJS Project](#start-nextjs-project)
- [Define all chartjs charts](#define-all-chartjs-charts)
- [Write a functional component](#write-a-functional-component)
- [Write a chartjs wrapper](#write-a-chartjs-wrapper)
- [Layout](#layout)
- [Optimize](#optimize)
- [Summary](#summary)

<!--te-->

# Demo(seeing is believing)

Before diving in, we will show you all charts I created.

![chartjs-react-chartjs-2-nextjs-tailwind-typescript-using-grid](https://user-images.githubusercontent.com/74223747/224673738-9edb7c6d-5e49-44a2-9bb0-00fa9eb5f608.png)

# Start NextJS Project

First, we'll create a nextjs + tailwind + typescript project using `npx`:

```bash
npx create-next-app@latest react-chartjs-2-nextjs-tailwind-example --typescript --eslint

cd react-chartjs-2-nextjs-tailwind-example
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

# Define all chartjs charts

We copy all chartjs official example codes and put them in `component/chartjsofficial` directory. We also define title and component for each example chart. The `components` array is the chart data we will render later.

```jsx
import { VerticalBarChart } from "@/component/chartjsofficial/verticalbarchart";
import { HorizontalBarChart } from "@/component/chartjsofficial/horizontalbarchart";
import { StackedBarChart } from "@/component/chartjsofficial/stackedbarchart";
import { GroupedBarChart } from "@/component/chartjsofficial/groupedbarchart";
import { AreaChart } from "@/component/chartjsofficial/areachart";
import { LineChart } from "@/component/chartjsofficial/linechart";
import { MultiAxisLineChart } from "@/component/chartjsofficial/multiaxislinechart";
import { PieChart } from "@/component/chartjsofficial/piechart";
import { DoughnutChart } from "@/component/chartjsofficial/doughnutchart";
import { PolarAreaChart } from "@/component/chartjsofficial/polarareachart";
import { RadarChart } from "@/component/chartjsofficial/radarchart";
import { ScatterChart } from "@/component/chartjsofficial/scatterchart";
import { BubbleChart } from "@/component/chartjsofficial/bubblechart";
import { MultiTypeChart } from "@/component/chartjsofficial/multitypechart";
import { ChartEvents } from "@/component/chartjsofficial/chartevents";
import { ChartRef } from "@/component/chartjsofficial/chartref";
import { GradientChart } from "@/component/chartjsofficial/gradientchart";
import { ChartEventsSingleDataset } from "@/component/chartjsofficial/charteventssingledataset";
import { ChartEventsSingleDatasetOutsideDatasource } from "@/component/chartjsofficial/charteventssingledatasetoutsidedatasource";

const components = [
  {
    title: "VerticalBarChart",
    component: VerticalBarChart,
  },
  {
    title: "HorizontalBarChart",
    component: HorizontalBarChart,
  },
  {
    title: "StackedBarChart",
    component: StackedBarChart,
  },
  {
    title: "GroupedBarChart",
    component: GroupedBarChart,
  },
  {
    title: "AreaChart",
    component: AreaChart,
  },
  {
    title: "LineChart",
    component: LineChart,
  },
  {
    title: "MultiAxisLineChart",
    component: MultiAxisLineChart,
  },
  {
    title: "DoughnutChart",
    component: DoughnutChart,
  },
  {
    title: "PolarAreaChart",
    component: PolarAreaChart,
  },
  {
    title: "RadarChart",
    component: RadarChart,
  },
  {
    title: "ScatterChart",
    component: ScatterChart,
  },
  {
    title: "BubbleChart",
    component: BubbleChart,
  },
  {
    title: "ScatterChart",
    component: ScatterChart,
  },
  {
    title: "MultiTypeChart",
    component: MultiTypeChart,
  },
  {
    title: "ChartEvents",
    component: ChartEvents,
  },
  {
    title: "ChartRef",
    component: ChartRef,
  },
  {
    title: "GradientChart",
    component: GradientChart,
  },
  {
    title: "ChartEventsSingleDataset",
    component: ChartEventsSingleDataset,
  },
];
```

# Write a functional component

In order to render all components(charts), we write a functional component called `ComponentWrapper` to take array of components with title and render them in one place.

```jsx
type Component = {
  title: string,
  component: React.FunctionComponent,
};
type ChartProps = {
  components: Component[],
};

const ComponentWrapper: React.FC<ChartProps> = ({ components }) => {
  return (
    <div>
      {components.map((component, index) => {
        return (
          <ChartWrapper key={index} title={component.title}>
            <component.component />
          </ChartWrapper>
        );
      })}
    </div>
  );
};
```

The `ComponentWrapper` component maps over the components array and renders each component wrapped in a `ChartWrapper` component(explained later).

It receives a prop object of type `ChartProps`, destructures the `components` property from that object, and uses it to render a list of React function components.

We can refer to the component part from item in `components` array, i.e. `ChartEventsSingleDataset`

```jsx
import React, { MouseEvent, useRef } from "react";
import type { InteractionItem } from "chart.js";
import {
  Chart as ChartJS,
  LinearScale,
  CategoryScale,
  BarElement,
  PointElement,
  LineElement,
  Legend,
  Tooltip,
} from "chart.js";
import {
  Chart,
  getDatasetAtEvent,
  getElementAtEvent,
  getElementsAtEvent,
} from "react-chartjs-2";
import faker from "faker";

ChartJS.register(
  LinearScale,
  CategoryScale,
  BarElement,
  PointElement,
  LineElement,
  Legend,
  Tooltip
);

export const options = {
  scales: {
    y: {
      beginAtZero: true,
    },
  },
};

const labels = ["January", "February", "March", "April", "May", "June", "July"];

export const data = {
  labels,
  datasets: [
    {
      type: "bar" as const,
      label: "Dataset 1",
      borderColor: "rgb(255, 99, 132)",
      backgroundColor: "rgb(255, 99, 132)",
      data: labels.map(() => faker.datatype.number({ min: -1000, max: 1000 })),
    },
  ],
};

export function ChartEventsSingleDataset() {
  const printDatasetAtEvent = (dataset: InteractionItem[]) => {
    if (!dataset.length) return;

    const datasetIndex = dataset[0].datasetIndex;

    console.log(`printDatasetAtEvent: ${data.datasets[datasetIndex].label}`);
  };

  const printElementAtEvent = (element: InteractionItem[]) => {
    if (!element.length) return;

    const { datasetIndex, index } = element[0];

    console.log(
      `index: ${data.labels[index]}, data: ${data.datasets[datasetIndex].data[index]}`
    );
  };

  const printElementsAtEvent = (elements: InteractionItem[]) => {
    if (!elements.length) return;

    console.log(`element length: ${elements.length}`);
  };

  const chartRef = useRef<ChartJS>(null);

  const onClick = (event: MouseEvent<HTMLCanvasElement>) => {
    const { current: chart } = chartRef;

    if (!chart) {
      return;
    }

    printDatasetAtEvent(getDatasetAtEvent(chart, event));
    printElementAtEvent(getElementAtEvent(chart, event));
    printElementsAtEvent(getElementsAtEvent(chart, event));
  };

  return (
    <Chart
      ref={chartRef}
      type="bar"
      onClick={onClick}
      options={options}
      data={data}
    />
  );
}
```

# Write a chartjs wrapper

In order to add title for each chart, we write a functional component to wrapper up every chart with a `h1`. Note we define the `h1` style using tailwindcss, setting div size(`h-96`) and text size(`text-3xl`), etc.

```jsx
export const ChartWrapper: React.FC<{
  title: string,
  children: React.ReactNode,
}> = ({ title, children }) => {
  return (
    <div className="max-h-96 h-96  bg-slate-50 border border-dashed">
      <h1 className="text-3xl text-center font-bold underline">{title}</h1>
      {children}
    </div>
  );
};
```

The `ChartWrapper` is also a functional component that renders `title` in `h1` tag and `children` property below it.

All other chartjs components are straightforward.

# Layout

Now, we need to define the layout of all charts. We can use `grid` layout from tailwind and put all charts into it.

In `App` component, we iterate the components and render them in a div with `grid` class.

```jsx
export default function App() {
  return (
    <div>
      <Head>
        <title>ChartJS + NextJS + TailwindCSS</title>
        <meta name="description" content="Generated by create next app" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="icon" href="/favicon.ico" />
      </Head>
      <main className={styles.main}>
        <div className={styles.description}>
          <h1 className="text-3xl text-center font-bold underline">
            ChartJS + NextJS + TailwindCSS
          </h1>
          <div className="container columns-2">
            <div className="grid grid-cols-2 md:grid-cols-1 gap-0">
              {<ComponentWrapper components={components} />}
            </div>
          </div>
        </div>
      </main>
    </div>
  );
}
```

This is a React component called App that is exporting as a default. The component renders a main `div` element that contains a `Head` component and a main element with a className of `styles.main`. Within the main element, there is a div element with a `className` of `styles.description` that contains a `h1` element with a `className` of `"text-3xl text-center font-bold underline"` and some text.

The div element also contains a div element with a `className` of `"container columns-2"` that contains an inner div element with a className of `"grid grid-cols-2 md:grid-cols-1 gap-0"`. This inner div element renders the `ComponentWrapper` component passing an array of components in the components prop.

Here is the important part:

```jsx
{
  <ComponentWrapper components={components} />;
}
```

The `ComponentWrapper` component dynamically renders each component passed in the `components` array. The rendering of each component is wrapped in a `ChartWrapper` component that provides a title for each component.

# Optimize

Can we do better? The answer is yes!

Notice, for `ComponentWrapper` function component, it's more verbose. We could write it in another way.

Here is the old way.

```jsx
// NOTE: OK but verbose
const ComponentWrapper: React.FC<ChartProps> = ({ components }) => {
  return (
    <div>
      {components.map((c, index) => {
        return (
          <ChartWrapper key={index} title={c.title}>
            <c.component />
          </ChartWrapper>
        );
      })}
    </div>
  );
};
```

The new way:

```jsx
// NOTE: concise
const ComponentWrapperNew: React.FC<ChartProps> = ({ components }) => {
  return (
    <div>
      {components.map((c, index) => {
        const { title, component: C } = c;

        return (
          <ChartWrapper key={index} title={title}>
            <C />
          </ChartWrapper>
        );
      })}
    </div>
  );
};
```

To render list of charts components with titles and properties, we wrote two React component functions. Although both functions produce the same output, their implementation approaches are different.

The first function `ComponentWrapper` is a good method, but the downside is that it is quite verbose. It uses curly braces and a return statement to wrap the component inside `<ChartWrapper>`, and it also references `c.title` and `c.component` multiple times within the function body.

The second function `ComponentWrapperNew` is a more concise implementation. It uses ES6 destructuring syntax to get `title` and `component` from the `components` object, which avoids repeated use of `c.title` and `c.component`. Additionally, it uses a more descriptive variable name `C` for the component. We can also call it `ChartComponent` if you like.

Clearly, the second implementation is more succinct, easier to read and maintain, and also makes it easier to add new properties or methods. Therefore, we recommend using the second function `ComponentWrapperNew` as a more efficient programming practice.

To summarize, the primary difference between these two implementation methods is that the first one is verbose, which makes it difficult to read and maintain, while the second one is more concise and easier to read and maintain. The second function also uses destructuring, which is a useful syntax for improving readability by eliminating repetition. By using a more concise and readable implementation, you can streamline your code and make it more efficient.

# Summary

In this post, we compare and contrast two React component functions that are used to render a list of chart components with titles and properties. While both functions ultimately produce the same result, their implementation approaches are different.
