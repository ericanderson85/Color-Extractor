<script>
import FileDrop from './FileDrop.svelte'
import Swatches from './Swatches.svelte'

const COLOR_BIN_SIZE = 32; // Size of the bins for color grouping
const SIMILARITY_THRESHOLD = 85; // Threshold for grouping colors into clusters
const MAX_DISPLAYED_COLORS = 5; // Maximum number of color swatches to display

let displayedColorSwatches = [];

// Coordinates the extraction of colors from the uploaded image
export function extractColorsFromImage(image) {
  const imageData = getImageData(image);
  const colorFrequencyMap = buildColorFrequencyMap(imageData);
  const colorList = mapToList(colorFrequencyMap);
  const colorClusters = clusterColors(colorList);
  const averagedColors = averageClusterColors(colorClusters);
  const sortedClusters = sortClustersByImportance(averagedColors);
  displayedColorSwatches = sortedClusters.slice(0, MAX_DISPLAYED_COLORS).map(cluster => ({
    color: cluster.color,
    hex: rgbToHex(cluster.color)
  }));
}

// Extracts and returns pixel data from an image by drawing it onto an off-screen canvas
function getImageData(image) {
  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d');
  canvas.width = image.width;
  canvas.height = image.height;
  context.drawImage(image, 0, 0);
  return context.getImageData(0, 0, canvas.width, canvas.height).data;
}

// Builds and returns a frequency map of colors from image data
function buildColorFrequencyMap(data) {
  const colorMap = {};
  for (let i = 0; i < data.length; i += 4) {
    const red = data[i];
    const green = data[i + 1];
    const blue = data[i + 2];
    const vibrance = calculateVibrance(red, green, blue);
    const colorKey = `${Math.floor(red / COLOR_BIN_SIZE) * COLOR_BIN_SIZE},${Math.floor(green / COLOR_BIN_SIZE) * COLOR_BIN_SIZE},${Math.floor(blue / COLOR_BIN_SIZE) * COLOR_BIN_SIZE}`;
    colorMap[colorKey] = colorMap[colorKey] || {
      count: 0,
      totalVibrance: 0
    };
    colorMap[colorKey].count++;
    colorMap[colorKey].totalVibrance += vibrance;
  }
  return colorMap;
}

// Converts the color frequency map to a list of color objects
function mapToList(colorMap) {
  return Object.keys(colorMap).map(key => {
    const [red, green, blue] = key.split(',').map(Number);
    const {
      count,
      totalVibrance
    } = colorMap[key];
    return {
      color: `rgb(${red},${green},${blue})`,
      count,
      vibrance: totalVibrance / count
    };
  });
}

// Groups colors into clusters based on their proximity in RGB space
function clusterColors(colors) {
  const clusters = [];
  colors.forEach(color => {
    let foundCluster = false;
    for (let cluster of clusters) {
      if (calculateColorDistance(cluster[0], color) < SIMILARITY_THRESHOLD) {
        cluster.push(color);
        foundCluster = true;
        break;
      }
    }
    if (!foundCluster) {
      clusters.push([color]);
    }
  });
  return clusters;
}

// Calculates the average color for each cluster
function averageClusterColors(clusters) {
  return clusters.map(cluster => {
    let totalRed = 0,
      totalGreen = 0,
      totalBlue = 0,
      totalCount = 0;
    cluster.forEach(color => {
      const [red, green, blue] = color.color.replace('rgb(', '').replace(')', '').split(',').map(Number);
      totalRed += red * color.count;
      totalGreen += green * color.count;
      totalBlue += blue * color.count;
      totalCount += color.count;
    });
    const averageRed = Math.round(totalRed / totalCount);
    const averageGreen = Math.round(totalGreen / totalCount);
    const averageBlue = Math.round(totalBlue / totalCount);
    const averageColor = `rgb(${averageRed},${averageGreen},${averageBlue})`;
    const averageVibrance = calculateVibrance(averageRed, averageGreen, averageBlue);

    return {
      color: averageColor,
      count: totalCount,
      vibrance: averageVibrance
    };
  });
}

// Sorts color clusters based on vibrance and count
function sortClustersByImportance(clusters) {
  return clusters.sort((a, b) => {
    const scoreA = a.vibrance ** 4 * a.count;
    const scoreB = b.vibrance ** 4 * b.count;
    return scoreB - scoreA;
  });
}

// Calculates and returns the vibrance of a color based on its RGB components
function calculateVibrance(red, green, blue) {
  const rNorm = red / 255;
  const gNorm = green / 255;
  const bNorm = blue / 255;
  const luminance = 0.2126 * rNorm + 0.7152 * gNorm + 0.0722 * bNorm;
  const maxValue = Math.max(rNorm, gNorm, bNorm);
  const minValue = Math.min(rNorm, gNorm, bNorm);
  const saturation = maxValue === 0 ? 0 : (maxValue - minValue) / maxValue;
  const vibrance = saturation * (luminance ** 0.5);
  return vibrance * 1.5;
}

// Calculates the Euclidean distance between two colors
function calculateColorDistance(firstColor, secondColor) {
  const [red1, green1, blue1] = firstColor.color.replace('rgb(', '').replace(')', '').split(',').map(Number);
  const [red2, green2, blue2] = secondColor.color.replace('rgb(', '').replace(')', '').split(',').map(Number);
  return Math.sqrt((red2 - red1) ** 2 + (green2 - green1) ** 2 + (blue2 - blue1) ** 2);
}

function rgbToHex(rgb) {
  let [r, g, b] = rgb.match(/\d+/g);
  return `#${[r, g, b].map(x => {
          const hex = parseInt(x).toString(16);
          return hex.length === 1 ? '0' + hex : hex;
      }).join('')}`;
}
</script>

<main>
    <div class="title-container">
        <span class="title"><a href="/">Color Palette Extractor</a></span>
        <span class="description">Get a color palette from an image!</span>
    </div>
        <FileDrop on:imageReady="{e => extractColorsFromImage(e.detail.imageElement)}"/>
        <Swatches displayedColorSwatches={displayedColorSwatches} />
    </main>

<style>
    a {
        text-decoration: none;
    }

    .title-container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 4px;
        margin:20px;
    }

    .title {
        font-size: 40px;
        font-weight: 600;
        text-align: center;
    }

    .description {
        font-size: 20px;
        text-align: center;
    }
</style>
