---
title: Label
order: 1
---
import DataSet from '@antv/data-set';
import { Chart } from '@antv/g2';

const data = [
  { country: 'Post 70s and older', year: 'LD 2019', value: 28 },
  { country: 'Post 70s and older', year: 'DF 2019', value: 27 },
  { country: 'Post 70s and older', year: 'MF 2019', value: 31 },
  { country: 'Post 70s and older', year: 'ND 2019', value: 9 },
  { country: 'Post 70s and older', year: 'LD 2020', value: 16 },
  { country: 'Post 70s and older', year: 'DF 2020', value: 15 },
  { country: 'Post 70s and older', year: 'ND 2020', value: 10 },
  { country: 'post-80s', year: 'LD 2019', value: 30 },
  { country: 'post-80s', year: 'DF 2019', value: 29 },
  { country: 'post-80s', year: 'MF 2019', value: 35 },
  { country: 'post-80s', year: 'ND 2019', value: 39 },
  { country: 'post-80s', year: 'LD 2020', value: 27 },
  { country: 'post-80s', year: 'DF 2020', value: 26 },
  { country: 'post-80s', year: 'ND 2020', value: 31 },
  { country: 'post-90s and younger', year: 'LD 2019', value: 42 },
  { country: 'post-90s and younger', year: 'DF 2019', value: 44 },
  { country: 'post-90s and younger', year: 'MF 2019', value: 34 },
  { country: 'post-90s and younger', year: 'ND 2019', value: 52 },
  { country: 'post-90s and younger', year: 'LD 2020', value: 57 },
  { country: 'post-90s and younger', year: 'DF 2020', value: 59 },
  { country: 'post-90s and younger', year: 'ND 2020', value: 59 },
];
const ds = new DataSet();
const dv = ds
  .createView('demo')
  .source(data)
  .transform({
    type: 'percent',
    field: 'value', // 统计销量
    dimension: 'country', // 每年的占比
    groupBy: ['year'], // 以不同产品类别为分组
    as: 'percent',
  });

const chart = new Chart({
  container: 'container',
  autoFit: true,
  height: 500,
});
chart.data(dv.rows);
chart.scale({
  percent: {
    min: 0,
    formatter(val) {
      return (val * 100).toFixed(0) + '%';
    },
  },
});
chart.tooltip(false);
chart
  .interval()
  .adjust('stack')
  .position('year*percent')
  .label('percent', {
    position: 'middle',
    offset: 0,
    style: {
      fill: '#fff',
      stroke: null
    },
  })
  .color('country');
chart.interaction('element-highlight-by-color');
chart.render();
function toDataURL(chart: Chart) {
  const canvas = chart.getCanvas();
  const renderer = chart.renderer;
  const canvasDom = canvas.get('el');
  let dataURL = '';
  if (renderer === 'svg') {
    const clone = canvasDom.cloneNode(true);
    const svgDocType = document.implementation.createDocumentType(
      'svg',
      '-//W3C//DTD SVG 1.1//EN',
      'http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd'
    );
    const svgDoc = document.implementation.createDocument('http://www.w3.org/2000/svg', 'svg', svgDocType);
    svgDoc.replaceChild(clone, svgDoc.documentElement);
    const svgData = new XMLSerializer().serializeToString(svgDoc);
    dataURL = 'data:image/svg+xml;charset=utf8,' + encodeURIComponent(svgData);
  } else if (renderer === 'canvas') {
    dataURL = canvasDom.toDataURL('image/png');
  }
  return dataURL;
}

/**
 * 图表图片导出
 * @param chart chart 实例
 * @param name 图片名称，可选，默认名为 'G2Chart'
 */
function downloadImage(chart: Chart, name: string = 'G2Chart') {
  const link = document.createElement('a');
  const renderer = chart.renderer;
  const filename = `${name}${renderer === 'svg' ? '.svg' : '.png'}`;
  const canvas = chart.getCanvas();
  canvas.get('timeline').stopAllAnimations();

  setTimeout(() => {
    const dataURL = toDataURL(chart);
    if (window.Blob && window.URL && renderer !== 'svg') {
      const arr = dataURL.split(',');
      const mime = arr[0].match(/:(.*?);/)[1];
      const bstr = atob(arr[1]);
      let n = bstr.length;
      const u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      const blobObj = new Blob([u8arr], { type: mime });
      if (window.navigator.msSaveBlob) {
        window.navigator.msSaveBlob(blobObj, filename);
      } else {
        link.addEventListener('click', () => {
          link.download = filename;
          link.href = window.URL.createObjectURL(blobObj);
        });
      }
    } else {
      link.addEventListener('click', () => {
        link.download = filename;
        link.href = dataURL;
      });
    }
    const e = document.createEvent('MouseEvents');
    e.initEvent('click', false, false);
    link.dispatchEvent(e);
  }, 16);
}
