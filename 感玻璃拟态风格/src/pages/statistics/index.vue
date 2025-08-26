<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any"
    class="min-h-screen bg-gradient-to-br from-indigo-900 via-purple-900 to-pink-900 relative overflow-hidden">

    <!-- Background Effects -->
    <div class="absolute inset-0 opacity-20">
      <div class="absolute top-20 left-20 w-72 h-72 bg-blue-400 rounded-full blur-3xl animate-pulse"></div>
      <div class="absolute top-40 right-32 w-96 h-96 bg-purple-400 rounded-full blur-3xl animate-pulse delay-1000">
      </div>
      <div class="absolute bottom-20 left-1/3 w-80 h-80 bg-pink-400 rounded-full blur-3xl animate-pulse delay-2000">
      </div>
    </div>

    <div class="relative z-10 p-6 space-y-6">
      <!-- Page Header -->
      <Motion :initial="{ opacity: 0, y: -30 }" :animate="{ opacity: 1, y: 0 }"
        :transition="{ duration: 0.8, delay: 0.1 }">
        <div class="backdrop-blur-xl bg-white/10 rounded-3xl border border-white/20 shadow-2xl p-8">
          <div class="max-w-7xl mx-auto">
            <h1
              class="text-4xl font-bold bg-gradient-to-r from-white via-blue-100 to-purple-200 bg-clip-text text-transparent mb-3">
              统计信息
            </h1>
            <p class="text-white/80 text-lg font-light">工地安全帽检测系统的详细数据分析和统计报告</p>
          </div>
        </div>
      </Motion>

      <!-- Statistics Overview -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.2 } as any">
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
          <div
            class="relative overflow-hidden backdrop-blur-md bg-gradient-to-br rounded-xl p-6 border transition-all duration-300 hover:scale-105 hover:shadow-lg from-blue-500/20 to-cyan-500/20 border-blue-400/30">
            <div class="text-3xl font-bold text-white mb-2">{{ computedStats.totalDetections.toLocaleString() }}</div>
            <div class="text-blue-200 text-sm font-medium">总检测次数</div>
          </div>
          <div
            class="relative overflow-hidden backdrop-blur-md bg-gradient-to-br rounded-xl p-6 border transition-all duration-300 hover:scale-105 hover:shadow-lg from-green-500/20 to-emerald-500/20 border-green-400/30">
            <div class="text-3xl font-bold text-white mb-2">{{ computedStats.complianceRate }}%</div>
            <div class="text-green-200 text-sm font-medium">安全帽佩戴率</div>
          </div>
          <div
            class="relative overflow-hidden backdrop-blur-md bg-gradient-to-br rounded-xl p-6 border transition-all duration-300 hover:scale-105 hover:shadow-lg from-orange-500/20 to-red-500/20 border-orange-400/30">
            <div class="text-3xl font-bold text-white mb-2">{{ computedStats.totalWarnings.toLocaleString() }}</div>
            <div class="text-orange-200 text-sm font-medium">警告总数</div>
          </div>
          <div
            class="relative overflow-hidden backdrop-blur-md bg-gradient-to-br rounded-xl p-6 border transition-all duration-300 hover:scale-105 hover:shadow-lg from-purple-500/20 to-pink-500/20 border-purple-400/30">
            <div class="text-3xl font-bold text-white mb-2">{{ computedStats.activeCameras }}</div>
            <div class="text-purple-200 text-sm font-medium">活跃摄像头</div>
          </div>
        </div>
      </Motion>

      <!-- Chart Grid -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.3 } as any">
        <div class="grid grid-cols-1 xl:grid-cols-2 gap-6">
          <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
            <h3 class="text-xl font-semibold text-white mb-4">📈 检测趋势分析</h3>
            <div class="h-80"><v-chart class="chart" :option="trendChartOption" autoresize /></div>
          </div>
          <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
            <h3 class="text-xl font-semibold text-white mb-4">⚠️ 警告级别分布</h3>
            <div class="h-80"><v-chart class="chart" :option="warningChartOption" autoresize /></div>
          </div>
          <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
            <h3 class="text-xl font-semibold text-white mb-4">📹 摄像头检测统计</h3>
            <div class="h-80"><v-chart class="chart" :option="cameraChartOption" autoresize /></div>
          </div>
          <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
            <h3 class="text-xl font-semibold text-white mb-4">🕐 时段违规分析</h3>
            <div class="h-80"><v-chart class="chart" :option="hourlyChartOption" autoresize /></div>
          </div>
        </div>
      </Motion>

      <!-- Detailed Data Table -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.4 } as any">
        <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
          <h3 class="text-xl font-semibold text-white mb-6">📋 详细统计数据</h3>
          <div class="overflow-x-auto custom-scrollbar">
            <table class="w-full text-sm text-left text-white/90">
              <thead class="text-xs text-white/70 uppercase bg-white/5">
                <tr>
                  <th scope="col" class="px-6 py-3">摄像头</th>
                  <th scope="col" class="px-6 py-3">检测次数</th>
                  <th scope="col" class="px-6 py-3">佩戴安全帽</th>
                  <th scope="col" class="px-6 py-3">未佩戴安全帽</th>
                  <th scope="col" class="px-6 py-3">合规率</th>
                  <th scope="col" class="px-6 py-3">警告次数</th>
                  <th scope="col" class="px-6 py-3">最后检测时间</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="camera in cameraStats" :key="camera.id"
                  class="border-b border-white/10 hover:bg-white/5 transition-colors duration-200">
                  <td class="px-6 py-4 font-medium">{{ camera.name }}</td>
                  <td class="px-6 py-4">{{ camera.detections.toLocaleString() }}</td>
                  <td class="px-6 py-4 text-green-300 font-semibold">{{ camera.withHelmet.toLocaleString() }}</td>
                  <td class="px-6 py-4 text-red-300 font-semibold">{{ camera.withoutHelmet.toLocaleString() }}</td>
                  <td class="px-6 py-4">
                    <span class="inline-flex items-center px-2 py-1 text-xs font-semibold rounded-full backdrop-blur-sm"
                      :class="getBadgeClass(camera.complianceRate)">
                      {{ camera.complianceRate }}%
                    </span>
                  </td>
                  <td class="px-6 py-4 text-orange-300 font-semibold">{{ camera.warnings }}</td>
                  <td class="px-6 py-4 text-white/60">{{ camera.lastDetection }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </Motion>

      <!-- Export Section -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.5 } as any">
        <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
          <h3 class="text-xl font-semibold text-white mb-6">📥 数据导出</h3>
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
            <button @click="exportExcel"
              class="relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-3 text-white/90 font-medium transition-all duration-300 transform hover:scale-105 from-green-500/20 to-emerald-500/20 border-green-400/30 hover:from-green-500/30 hover:to-emerald-500/30">📊
              导出Excel报表</button>
            <button @click="exportPDF"
              class="relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-3 text-white/90 font-medium transition-all duration-300 transform hover:scale-105 from-purple-500/20 to-pink-500/20 border-purple-400/30 hover:from-purple-500/30 hover:to-pink-500/30">📄
              导出PDF报告</button>
            <button @click="exportCSV"
              class="relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-3 text-white/90 font-medium transition-all duration-300 transform hover:scale-105 from-orange-500/20 to-red-500/20 border-orange-400/30 hover:from-orange-500/30 hover:to-red-500/30">📋
              导出CSV数据</button>
            <button @click="generateWeeklyReport"
              class="relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-3 text-white/90 font-medium transition-all duration-300 transform hover:scale-105 from-blue-500/20 to-cyan-500/20 border-blue-400/30 hover:from-blue-500/30 hover:to-cyan-500/30">📈
              生成周报</button>
          </div>
        </div>
      </Motion>
    </div>
  </Motion>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, watchEffect } from 'vue'
import { Motion } from 'motion-v'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart, PieChart, BarChart } from 'echarts/charts'
import { TitleComponent, TooltipComponent, LegendComponent, GridComponent } from 'echarts/components'
import VChart from 'vue-echarts'
import type { EChartsOption } from 'echarts'
import { getDetectionStats, getCameraStats } from '@/api/inspection/API'
import type { DailyStats, CameraStats as APICameraStats } from '@/types/apis/inspection_T'
import { jsPDF } from 'jspdf'

// ECharts setup
use([CanvasRenderer, LineChart, PieChart, BarChart, TitleComponent, TooltipComponent, LegendComponent, GridComponent]);

// Type Definitions
interface CameraStats {
  id: string; name: string; detections: number; withHelmet: number; withoutHelmet: number; complianceRate: number; warnings: number; lastDetection: string;
}

// Animation Variants
const pageVariants = { initial: { opacity: 0, y: 40 }, animate: { opacity: 1, y: 0 }, transition: { duration: 0.8, ease: 'easeOut' } };
const cardVariants = { initial: { opacity: 0, scale: 0.95, y: 20 }, animate: { opacity: 1, scale: 1, y: 0 }, transition: { duration: 0.6, ease: 'easeOut' } };

// Reactive Data
const cameraStats = ref<CameraStats[]>([]);
const dailyStats = ref<DailyStats[]>([]);
const loading = ref(false);

// Computed Stats
const computedStats = computed(() => {
  const totalDetections = cameraStats.value.reduce((sum, cam) => sum + cam.detections, 0);
  const totalWithHelmet = cameraStats.value.reduce((sum, cam) => sum + cam.withHelmet, 0);
  const totalWarnings = cameraStats.value.reduce((sum, cam) => sum + cam.withoutHelmet, 0);
  const complianceRate = totalDetections > 0 ? Number(((totalWithHelmet / totalDetections) * 100).toFixed(1)) : 0;
  return { totalDetections, complianceRate, totalWarnings, activeCameras: cameraStats.value.length };
});

// ECharts Dark Theme for Glassmorphism UI
const darkChartTheme = {
  backgroundColor: 'transparent',
  textStyle: { color: 'rgba(255, 255, 255, 0.9)' },
  title: { textStyle: { color: 'rgba(255, 255, 255, 0.9)', fontSize: 18 } },
  legend: { textStyle: { color: 'rgba(255, 255, 255, 0.7)' } },
  tooltip: { backgroundColor: 'rgba(20, 20, 30, 0.85)', borderColor: 'rgba(255, 255, 255, 0.2)', borderWidth: 1, textStyle: { color: '#FFF' } },
  xAxis: { axisLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.3)' } }, axisLabel: { color: 'rgba(255, 255, 255, 0.7)' }, splitLine: { show: false } },
  yAxis: { axisLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.3)' } }, axisLabel: { color: 'rgba(255, 255, 255, 0.7)' }, splitLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.2)' } } }
};

// Chart Options
const trendChartOption = ref<EChartsOption>({});
const warningChartOption = ref<EChartsOption>({});
const cameraChartOption = ref<EChartsOption>({});
const hourlyChartOption = ref<EChartsOption>({});

// Methods
const getBadgeClass = (rate: number) => {
  if (rate >= 95) return 'bg-green-500/30 text-green-200 border border-green-400/40';
  if (rate >= 90) return 'bg-amber-500/30 text-amber-200 border border-amber-400/40';
  return 'bg-red-500/30 text-red-200 border border-red-400/40';
};

const loadStatsData = async () => {
  loading.value = true;
  try {
    const [detectionStatsRes, cameraStatsRes] = await Promise.all([getDetectionStats(), getCameraStats()]);
    if (detectionStatsRes.code === 200) dailyStats.value = detectionStatsRes.data.daily_stats;
    if (cameraStatsRes.code === 200) {
      cameraStats.value = cameraStatsRes.data.camera_stats.map((cam: APICameraStats) => ({
        id: cam.camera_id, name: `${cam.camera_name}`, detections: cam.total_detections,
        withHelmet: cam.wearing_hat_count, withoutHelmet: cam.violation_count,
        complianceRate: Number((100 - cam.violation_rate).toFixed(1)), warnings: cam.violation_count, // Use total violations as warnings
        lastDetection: new Date(cam.latest_detection_time).toLocaleString('zh-CN', { hour12: false })
      }));
    }
  } catch (error) { console.error('加载统计数据失败:', error); }
  finally { loading.value = false; }
};

watchEffect(() => {
  const dates = dailyStats.value.map(item => item.date.slice(5));
  const totalData = dailyStats.value.map(item => item.total);
  const violationData = dailyStats.value.map(item => item.no_hat);
  const complianceData = dailyStats.value.map(item => item.total > 0 ? Number(((item.wearing_hat / item.total) * 100).toFixed(1)) : 0);

  trendChartOption.value = { ...darkChartTheme, legend: { ...darkChartTheme.legend, data: ['总检测数', '违规数', '合规率'], top: 30 }, grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true }, xAxis: { ...darkChartTheme.xAxis, type: 'category', data: dates }, yAxis: [{ ...darkChartTheme.yAxis, type: 'value', name: '检测数' }, { ...darkChartTheme.yAxis, type: 'value', name: '合规率(%)', max: 100 }], series: [{ name: '总检测数', type: 'line', data: totalData, smooth: true, itemStyle: { color: '#38BDF8' } }, { name: '违规数', type: 'line', data: violationData, smooth: true, itemStyle: { color: '#F87171' } }, { name: '合规率', type: 'line', yAxisIndex: 1, data: complianceData, smooth: true, itemStyle: { color: '#34D399' } }] };

  const cameraNames = cameraStats.value.map(cam => cam.name);
  const camDetections = cameraStats.value.map(cam => cam.detections);
  const camViolations = cameraStats.value.map(cam => cam.withoutHelmet);

  cameraChartOption.value = { ...darkChartTheme, legend: { ...darkChartTheme.legend, data: ['检测总数', '违规数'], top: 30 }, grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true }, xAxis: { ...darkChartTheme.xAxis, type: 'category', data: cameraNames }, yAxis: { ...darkChartTheme.yAxis, type: 'value' }, series: [{ name: '检测总数', type: 'bar', data: camDetections, itemStyle: { color: '#60A5FA' } }, { name: '违规数', type: 'bar', data: camViolations, itemStyle: { color: '#F472B6' } }] };

  warningChartOption.value = { ...darkChartTheme, tooltip: { trigger: 'item', formatter: '{a} <br/>{b}: {c} ({d}%)' }, legend: { ...darkChartTheme.legend, orient: 'vertical', left: 'left', top: 'middle' }, series: [{ name: '检测结果分布', type: 'pie', radius: ['40%', '70%'], center: ['60%', '50%'], avoidLabelOverlap: false, label: { show: false }, emphasis: { label: { show: true, fontSize: '20', fontWeight: 'bold' } }, data: [{ value: computedStats.value.totalWarnings, name: '未佩戴安全帽', itemStyle: { color: '#F43F5E' } }, { value: computedStats.value.totalDetections - computedStats.value.totalWarnings, name: '佩戴安全帽', itemStyle: { color: '#10B981' } }] }] };

  hourlyChartOption.value = { ...darkChartTheme, tooltip: { trigger: 'axis', formatter: (p: any) => `${p[0].name}<br/>违规数: ${p[0].value}` }, grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true }, xAxis: { ...darkChartTheme.xAxis, type: 'category', data: ['08:00', '10:00', '12:00', '14:00', '16:00', '18:00'] }, yAxis: { ...darkChartTheme.yAxis, type: 'value', name: '违规数' }, series: [{ name: '违规数', type: 'bar', data: [15, 12, 20, 18, 25, 22], itemStyle: { color: '#F97316' } }] };
});

const createAndDownloadFile = (content: string, filename: string, type: string) => {
  const blob = new Blob(['\uFEFF' + content], { type });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = `${filename}_${new Date().toISOString().slice(0, 10)}.csv`;
  link.click();
  URL.revokeObjectURL(link.href);
};

const exportCSV = (): void => {
  if (cameraStats.value.length === 0) {
    alert('暂无统计数据可导出');
    return;
  }
  const headers = ['摄像头名称', '检测次数', '佩戴安全帽', '未佩戴安全帽', '合规率(%)', '警告次数', '最后检测时间'];
  const data = cameraStats.value.map(camera => [
    camera.name.replace('📹 ', '').split(' - ')[0],
    camera.detections,
    camera.withHelmet,
    camera.withoutHelmet,
    camera.complianceRate,
    camera.warnings,
    camera.lastDetection
  ]);
  const csvContent = [headers, ...data].map(row => row.map(cell => `"${String(cell).replace(/"/g, '""')}"`).join(',')).join('\n');
  createAndDownloadFile(csvContent, '统计数据', 'text/csv;charset=utf-8');
};

const exportExcel = (): void => {
  // Excel is often just CSV with a different name for basic exports
  exportCSV();
};

const exportPDF = async (): Promise<void> => {
  try {
    if (cameraStats.value.length === 0) {
      alert('暂无统计数据可生成PDF报告');
      return;
    }
    const doc = new jsPDF();
    const currentDate = new Date().toLocaleDateString('zh-CN');
    const totalStats = computedStats.value;

    doc.setFont('helvetica');
    doc.setFontSize(20);
    doc.text('Safety Helmet Detection Report', 105, 20, { align: 'center' });
    doc.setFontSize(12);
    doc.text(`Report Date: ${currentDate}`, 105, 40, { align: 'center' });

    doc.setFontSize(14);
    doc.text('Statistics Overview', 20, 60);

    doc.setFontSize(10);
    let yPos = 75;
    doc.text(`Total Detections: ${totalStats.totalDetections.toLocaleString()}`, 20, yPos); yPos += 10;
    doc.text(`Compliance Rate: ${totalStats.complianceRate}%`, 20, yPos); yPos += 10;
    doc.text(`Total Warnings: ${totalStats.totalWarnings.toLocaleString()}`, 20, yPos); yPos += 10;
    doc.text(`Active Cameras: ${totalStats.activeCameras}`, 20, yPos); yPos += 20;

    doc.setFontSize(14);
    doc.text('Camera Statistics Details', 20, yPos); yPos += 15;

    // Manual table drawing (original logic restored)
    doc.setFontSize(9);
    const headers = ['Camera', 'Detections', 'With Helmet', 'Without Helmet', 'Compliance%', 'Warnings', 'Last Detection'];
    const colWidths = [30, 20, 20, 25, 20, 20, 35];
    let xPos = 20;

    doc.setFillColor(240, 240, 240);
    doc.rect(20, yPos - 5, 170, 10, 'F');
    headers.forEach((header, index) => {
      doc.text(header, xPos + 2, yPos);
      xPos += colWidths[index];
    });
    yPos += 10;

    cameraStats.value.forEach((camera, index) => {
      if (yPos > 270) {
        doc.addPage();
        yPos = 20;
      }
      xPos = 20;
      const rowData = [
        camera.name.length > 12 ? camera.name.substring(0, 12) + '...' : camera.name,
        camera.detections.toString(), camera.withHelmet.toString(), camera.withoutHelmet.toString(),
        camera.complianceRate + '%', camera.warnings.toString(),
        camera.lastDetection.length > 15 ? camera.lastDetection.substring(0, 15) + '...' : camera.lastDetection
      ];
      if (index % 2 === 0) {
        doc.setFillColor(250, 250, 250);
        doc.rect(20, yPos - 5, 170, 10, 'F');
      }
      rowData.forEach((data, colIndex) => {
        doc.text(data, xPos + 2, yPos);
        xPos += colWidths[colIndex];
      });
      yPos += 10;
    });

    const pageCount = (doc as any).internal.getNumberOfPages();
    for (let i = 1; i <= pageCount; i++) {
      doc.setPage(i);
      doc.setFontSize(8);
      doc.text(`Generated by Safety Helmet Detection System - Page ${i}/${pageCount}`, 105, 290, { align: 'center' });
    }

    doc.save(`Safety_Helmet_Report_${new Date().toISOString().slice(0, 10)}.pdf`);
    alert('PDF报告生成成功！');
  } catch (error) {
    console.error('生成PDF报告失败:', error);
    alert('生成PDF报告失败，请重试');
  }
};

const generateWeeklyReport = (): void => {
  if (cameraStats.value.length === 0 && dailyStats.value.length === 0) {
    alert('暂无数据可生成周报');
    return;
  }
  const currentDate = new Date();
  const weekStart = new Date(currentDate.getTime() - 6 * 24 * 60 * 60 * 1000);
  const totalStats = computedStats.value;
  const weeklyAvgDetections = Math.round(totalStats.totalDetections / 7);
  const weeklyAvgWarnings = Math.round(totalStats.totalWarnings / 7);

  let htmlContent = `
    <!DOCTYPE html><html><head><meta charset="UTF-8"><title>安全帽检测周报</title><style>body { font-family: 'Microsoft YaHei', Arial, sans-serif; margin: 20px; line-height: 1.6; } .header { text-align: center; margin-bottom: 30px; border-bottom: 2px solid #333; padding-bottom: 20px; } .section { margin-bottom: 30px; } .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; margin: 20px 0; } .stat-card { border: 1px solid #ddd; padding: 15px; text-align: center; border-radius: 8px; background: #f9f9f9; } .stat-value { font-size: 24px; font-weight: bold; color: #333; } .stat-label { color: #666; margin-top: 5px; } .summary { background: #e8f4fd; padding: 20px; border-radius: 8px; margin: 20px 0; } .recommendations { background: #fff3cd; padding: 20px; border-radius: 8px; margin: 20px 0; } table { width: 100%; border-collapse: collapse; margin: 15px 0; } th, td { border: 1px solid #ddd; padding: 10px; text-align: left; } th { background-color: #f5f5f5; font-weight: bold; } .footer { margin-top: 40px; text-align: center; color: #666; border-top: 1px solid #ddd; padding-top: 20px; } .highlight { color: #d63384; font-weight: bold; } .good { color: #198754; font-weight: bold; }</style></head><body><div class="header"><h1>🏗️ 工地安全帽检测系统周报</h1><p>报告周期: ${weekStart.toLocaleDateString('zh-CN')} - ${currentDate.toLocaleDateString('zh-CN')}</p><p>报告生成时间: ${new Date().toLocaleString('zh-CN')}</p></div><div class="section"><h2>📊 本周统计概览</h2><div class="stats-grid"><div class="stat-card"><div class="stat-value">${totalStats.totalDetections.toLocaleString()}</div><div class="stat-label">总检测次数</div></div><div class="stat-card"><div class="stat-value ${totalStats.complianceRate >= 95 ? 'good' : totalStats.complianceRate >= 90 ? '' : 'highlight'}">${totalStats.complianceRate}%</div><div class="stat-label">安全帽佩戴率</div></div><div class="stat-card"><div class="stat-value ${totalStats.totalWarnings > 50 ? 'highlight' : ''}">${totalStats.totalWarnings.toLocaleString()}</div><div class="stat-label">警告总数</div></div></div></div><div class="section"><h2>📈 趋势分析</h2><div class="summary"><h3>本周关键指标</h3><ul><li>日均检测次数: <strong>${weeklyAvgDetections.toLocaleString()}</strong> 次</li><li>日均警告次数: <strong>${weeklyAvgWarnings.toLocaleString()}</strong> 次</li><li>活跃摄像头数量: <strong>${totalStats.activeCameras}</strong> 个</li><li>整体合规率: <strong class="${totalStats.complianceRate >= 95 ? 'good' : totalStats.complianceRate >= 90 ? '' : 'highlight'}">${totalStats.complianceRate}%</strong></li></ul></div></div><div class="section"><h2>📹 摄像头性能分析</h2><table><thead><tr><th>摄像头</th><th>检测次数</th><th>合规率</th><th>警告次数</th><th>状态评估</th></tr></thead><tbody>`;

  cameraStats.value.forEach(camera => {
    let status = '正常', statusClass = 'good';
    if (camera.complianceRate < 90) { status = '需关注'; statusClass = 'highlight'; }
    else if (camera.complianceRate < 95) { status = '一般'; statusClass = ''; }
    htmlContent += `<tr><td>${camera.name}</td><td>${camera.detections.toLocaleString()}</td><td class="${camera.complianceRate >= 95 ? 'good' : camera.complianceRate >= 90 ? '' : 'highlight'}">${camera.complianceRate}%</td><td>${camera.warnings}</td><td class="${statusClass}">${status}</td></tr>`;
  });

  const lowComplianceCameras = cameraStats.value.filter(c => c.complianceRate < 90);
  const highWarningCameras = cameraStats.value.filter(c => c.warnings > 10);

  htmlContent += `</tbody></table></div><div class="section"><h2>💡 安全建议与改进措施</h2><div class="recommendations"><h3>本周发现的问题:</h3><ul>`;

  if (totalStats.complianceRate < 90) htmlContent += `<li class="highlight">整体合规率偏低(${totalStats.complianceRate}%)，需要加强安全教育</li>`;
  if (lowComplianceCameras.length > 0) htmlContent += `<li class="highlight">以下区域合规率较低: ${lowComplianceCameras.map(c => c.name.split(' - ')[0]).join(', ')}</li>`;
  if (highWarningCameras.length > 0) htmlContent += `<li class="highlight">以下摄像头警告频繁: ${highWarningCameras.map(c => c.name.split(' - ')[0]).join(', ')}</li>`;
  if (totalStats.complianceRate >= 95 && lowComplianceCameras.length === 0) htmlContent += `<li class="good">整体安全状况良好，继续保持</li>`;

  htmlContent += `</ul><h3>改进建议:</h3><ul><li>定期组织安全帽佩戴规范培训</li><li>在违规高发区域增设安全提醒标识</li><li>建立安全检查奖惩机制</li><li>优化摄像头覆盖范围，消除监控盲区</li></ul></div></div><div class="footer"><p>本周报由工地安全帽检测系统自动生成</p></div></body></html>`;

  createAndDownloadFile(htmlContent, '安全帽检测周报', 'text/html;charset=utf-8');
  alert('周报已生成，请在浏览器中查看详细内容');
};

onMounted(loadStatsData);
</script>

<style scoped>
.chart {
  height: 100%;
  width: 100%;
}

.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}
</style>
