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

    <!-- Floating Header -->
    <Motion :initial="{ opacity: 0, y: -30 }" :animate="{ opacity: 1, y: 0 }"
      :transition="{ duration: 0.8, delay: 0.1 }">
      <div class="relative z-10 p-8">
        <div class="backdrop-blur-xl bg-white/10 rounded-3xl border border-white/20 shadow-2xl p-8 max-w-7xl mx-auto">
          <div class="flex items-center justify-between">
            <div>
              <h1
                class="text-4xl font-bold bg-gradient-to-r from-white via-blue-100 to-purple-200 bg-clip-text text-transparent mb-3">
                AI Safety Monitor
              </h1>
              <p class="text-white/80 text-lg font-light">Real-time helmet detection powered by YOLOv11 neural networks
              </p>
            </div>
            <div class="hidden md:flex items-center space-x-4">
              <div class="w-3 h-3 bg-green-400 rounded-full animate-ping"></div>
              <span class="text-white/90 font-medium">System Online</span>
            </div>
          </div>
        </div>
      </div>
    </Motion>

    <div class="relative z-10 max-w-7xl mx-auto px-8 pb-8">
      <div class="grid grid-cols-12 gap-6">

        <!-- Main Content Area -->
        <div class="col-span-12 xl:col-span-9 space-y-6">

          <!-- Control Panel -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.2 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <div class="flex items-center justify-between mb-6">
                <h2 class="text-xl font-semibold text-white flex items-center">
                  <div class="w-2 h-2 bg-cyan-400 rounded-full mr-3 animate-pulse"></div>
                  Control Center
                </h2>
                <div class="flex items-center space-x-2 text-white/70">
                  <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                  </svg>
                  <span class="text-sm">Active</span>
                </div>
              </div>

              <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4">
                <button @click="requestCameraPermission"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-emerald-500/20 to-teal-500/20 hover:from-emerald-500/30 hover:to-teal-500/30 border border-emerald-400/30 rounded-xl px-4 py-3 text-white transition-all duration-300 transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed"
                  :disabled="permissionRequested">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-emerald-400/10 to-teal-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z">
                      </path>
                    </svg>
                    <span class="text-sm font-medium">{{ permissionRequested ? 'Authorized' : 'Camera Access' }}</span>
                  </div>
                </button>

                <button @click="toggleAllCameras"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-blue-500/20 to-indigo-500/20 hover:from-blue-500/30 hover:to-indigo-500/30 border border-blue-400/30 rounded-xl px-4 py-3 text-white transition-all duration-300 transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed"
                  :disabled="!permissionRequested">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-blue-400/10 to-indigo-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path d="M2 6a2 2 0 012-2h6l2 2h6a2 2 0 012 2v6a2 2 0 01-2 2H4a2 2 0 01-2-2V6z"></path>
                    </svg>
                    <span class="text-sm font-medium">{{ allCamerasActive ? 'Stop All' : 'Start All' }}</span>
                  </div>
                </button>

                <button @click="toggleRenderMode"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-purple-500/20 to-pink-500/20 hover:from-purple-500/30 hover:to-pink-500/30 border border-purple-400/30 rounded-xl px-4 py-3 text-white transition-all duration-300 transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed"
                  :disabled="!permissionRequested">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-purple-400/10 to-pink-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M4 2a2 2 0 00-2 2v11a2 2 0 002 2h12a2 2 0 002-2V4a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z">
                      </path>
                    </svg>
                    <span class="text-sm font-medium">{{ renderMode === 'canvas' ? 'Video Mode' : 'Canvas Mode'
                      }}</span>
                  </div>
                </button>

                <button @click="exportWarnings"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-amber-500/20 to-orange-500/20 hover:from-amber-500/30 hover:to-orange-500/30 border border-amber-400/30 rounded-xl px-4 py-3 text-white transition-all duration-300 transform hover:scale-105">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-amber-400/10 to-orange-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z">
                      </path>
                    </svg>
                    <span class="text-sm font-medium">Export CSV</span>
                  </div>
                </button>

                <button @click="resetWarnings"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-red-500/20 to-rose-500/20 hover:from-red-500/30 hover:to-rose-500/30 border border-red-400/30 rounded-xl px-4 py-3 text-white transition-all duration-300 transform hover:scale-105">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-red-400/10 to-rose-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1z">
                      </path>
                    </svg>
                    <span class="text-sm font-medium">Reset</span>
                  </div>
                </button>
              </div>
            </div>
          </Motion>

          <!-- Camera Grid -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.3 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <div class="flex items-center justify-between mb-6">
                <h2 class="text-xl font-semibold text-white flex items-center">
                  <div class="w-2 h-2 bg-green-400 rounded-full mr-3 animate-pulse"></div>
                  Camera Network
                </h2>
                <div class="flex items-center space-x-4 text-white/70">
                  <span class="text-sm">{{cameras.filter(c => c.active).length}} / {{ cameras.length }} Active</span>
                </div>
              </div>

              <div v-if="cameras.length > 0" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <div v-for="camera in cameras" :key="camera.id"
                  class="group relative backdrop-blur-md bg-white/5 rounded-xl border border-white/20 overflow-hidden aspect-video cursor-pointer transition-all duration-500 hover:bg-white/10 hover:border-white/30 hover:shadow-2xl hover:scale-105"
                  @click="openPreview(camera)">

                  <!-- Camera Label -->
                  <div class="absolute top-3 left-3 z-10">
                    <div class="backdrop-blur-md bg-black/30 rounded-lg px-3 py-1 border border-white/20">
                      <span class="text-white text-sm font-medium">{{ camera.label || `Camera #${camera.id}` }}</span>
                    </div>
                  </div>

                  <!-- Status Indicator -->
                  <div class="absolute top-3 right-3 z-10">
                    <div class="flex items-center space-x-2">
                      <div class="w-2 h-2 rounded-full"
                        :class="camera.active ? 'bg-green-400 animate-pulse' : 'bg-gray-400'"></div>
                      <span class="text-white/80 text-xs">{{ camera.active ? 'LIVE' : 'OFF' }}</span>
                    </div>
                  </div>

                  <!-- Canvas Render Mode -->
                  <canvas v-if="camera.active && renderMode === 'canvas'"
                    :ref="(el) => setCanvasRef(camera.id, el as HTMLCanvasElement)"
                    class="w-full h-full object-cover rounded-xl"></canvas>

                  <!-- Video Render Mode -->
                  <video v-if="camera.active && renderMode === 'video'"
                    :ref="(el) => setVideoRef(camera.id, el as HTMLVideoElement)"
                    class="w-full h-full object-cover rounded-xl" autoplay muted playsinline></video>

                  <!-- Inactive State -->
                  <div v-else class="flex items-center justify-center h-full">
                    <div class="text-center text-white/50">
                      <svg class="w-16 h-16 mx-auto mb-3 opacity-40" fill="currentColor" viewBox="0 0 20 20">
                        <path
                          d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z">
                        </path>
                      </svg>
                      <p class="text-sm font-light">Camera Offline</p>
                    </div>
                  </div>

                  <!-- Control Button -->
                  <div class="absolute bottom-3 right-3 z-10">
                    <button @click.stop="toggleCamera(camera.id)"
                      class="backdrop-blur-md border border-white/30 rounded-lg px-3 py-1 text-xs font-medium transition-all duration-300 hover:scale-110"
                      :class="camera.active ? 'bg-red-500/20 text-red-200 hover:bg-red-500/30' : 'bg-green-500/20 text-green-200 hover:bg-green-500/30'"
                      :disabled="!permissionRequested">
                      {{ camera.active ? 'Stop' : 'Start' }}
                    </button>
                  </div>

                  <!-- Hover Overlay -->
                  <div
                    class="absolute inset-0 bg-gradient-to-t from-black/20 via-transparent to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300 rounded-xl pointer-events-none">
                  </div>
                </div>
              </div>
            </div>
          </Motion>

          <!-- Statistics Dashboard -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.4 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <h2 class="text-xl font-semibold text-white mb-6 flex items-center">
                <div class="w-2 h-2 bg-blue-400 rounded-full mr-3 animate-pulse"></div>
                Real-time Analytics
              </h2>

              <div class="grid grid-cols-2 lg:grid-cols-4 gap-6">
                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-blue-500/20 to-cyan-500/20 rounded-xl p-6 border border-blue-400/30 transition-all duration-300 hover:scale-105 hover:from-blue-500/30 hover:to-cyan-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-blue-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.onlinePeople }}</div>
                    <div class="text-blue-200 text-sm font-medium">Personnel Online</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-green-500/20 to-emerald-500/20 rounded-xl p-6 border border-green-400/30 transition-all duration-300 hover:scale-105 hover:from-green-500/30 hover:to-emerald-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-green-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.withHelmet }}</div>
                    <div class="text-green-200 text-sm font-medium">With Helmet</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-red-500/20 to-rose-500/20 rounded-xl p-6 border border-red-400/30 transition-all duration-300 hover:scale-105 hover:from-red-500/30 hover:to-rose-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-red-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.withoutHelmet }}</div>
                    <div class="text-red-200 text-sm font-medium">Without Helmet</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-amber-500/20 to-orange-500/20 rounded-xl p-6 border border-amber-400/30 transition-all duration-300 hover:scale-105 hover:from-amber-500/30 hover:to-orange-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-amber-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold mb-2" :class="{
                      'text-green-300': stats.complianceRate >= 90,
                      'text-amber-300': stats.complianceRate >= 70,
                      'text-red-300': stats.complianceRate < 70
                    }">
                      {{ stats.complianceRate }}%
                    </div>
                    <div class="text-amber-200 text-sm font-medium">Compliance Rate</div>
                  </div>
                </div>
              </div>
            </div>
          </Motion>
        </div>

        <!-- Sidebar -->
        <div class="col-span-12 xl:col-span-3 space-y-6">

          <!-- Real-time Alerts -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.5 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <div class="flex items-center justify-between mb-6">
                <h3 class="text-lg font-semibold text-white flex items-center">
                  <div class="w-2 h-2 bg-red-400 rounded-full mr-3 animate-ping"></div>
                  Live Alerts
                </h3>
                <div
                  class="w-6 h-6 rounded-full bg-red-500/20 border border-red-400/30 flex items-center justify-center">
                  <span class="text-xs text-red-300 font-bold">{{ warnings.length }}</span>
                </div>
              </div>

              <div class="space-y-3 max-h-80 overflow-y-auto custom-scrollbar">
                <div v-for="warning in warnings" :key="warning.id"
                  class="group relative backdrop-blur-md rounded-xl p-4 border transition-all duration-300 hover:scale-[1.02]"
                  :class="getWarningLevelInfo(warning.warning_level).cardClass">

                  <div class="flex items-start justify-between">
                    <div class="flex-1">
                      <div class="flex items-center gap-2 mb-2">
                        <div class="rounded-full px-2 py-1 text-xs font-semibold backdrop-blur-sm"
                          :class="getWarningLevelInfo(warning.warning_level).badgeClass">
                          {{ getWarningLevelInfo(warning.warning_level).text }}
                        </div>
                        <span class="text-xs text-white/60 font-medium">{{ warning.camera_name }}</span>
                      </div>
                      <p class="text-sm text-white/90 font-medium mb-2">{{ warning.title }}</p>
                      <p class="text-xs text-white/50">{{ formatDateTime(warning.created_at) }}</p>
                    </div>
                  </div>

                  <!-- Accent bar -->
                  <div class="absolute left-0 top-0 bottom-0 w-1 rounded-l-xl"
                    :class="getWarningLevelInfo(warning.warning_level).accentClass"></div>
                </div>
              </div>
            </div>
          </Motion>

          <!-- Quick Actions -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.6 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <h3 class="text-lg font-semibold text-white mb-6 flex items-center">
                <div class="w-2 h-2 bg-purple-400 rounded-full mr-3 animate-pulse"></div>
                Quick Actions
              </h3>

              <div class="space-y-3">
                <button @click="toggleAllCameras"
                  class="group w-full relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-slate-600/20 to-gray-600/20 hover:from-slate-600/30 hover:to-gray-600/30 border border-slate-500/30 rounded-xl p-4 text-white transition-all duration-300 transform hover:scale-[1.02] disabled:opacity-50 disabled:cursor-not-allowed"
                  :disabled="!permissionRequested">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-slate-400/10 to-gray-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center space-x-3">
                    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                      <path d="M2 6a2 2 0 012-2h6l2 2h6a2 2 0 012 2v6a2 2 0 01-2 2H4a2 2 0 01-2-2V6z"></path>
                    </svg>
                    <span class="font-medium">{{ allCamerasActive ? 'Stop All Cameras' : 'Start All Cameras' }}</span>
                  </div>
                </button>

                <button @click="exportWarnings"
                  class="group w-full relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-blue-600/20 to-indigo-600/20 hover:from-blue-600/30 hover:to-indigo-600/30 border border-blue-500/30 rounded-xl p-4 text-white transition-all duration-300 transform hover:scale-[1.02]">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-blue-400/10 to-indigo-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center space-x-3">
                    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z">
                      </path>
                    </svg>
                    <span class="font-medium">Export Alert Report</span>
                  </div>
                </button>

                <button @click="resetWarnings"
                  class="group w-full relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-orange-600/20 to-red-600/20 hover:from-orange-600/30 hover:to-red-600/30 border border-orange-500/30 rounded-xl p-4 text-white transition-all duration-300 transform hover:scale-[1.02]">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-orange-400/10 to-red-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center space-x-3">
                    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1z">
                      </path>
                    </svg>
                    <span class="font-medium">Clear All Alerts</span>
                  </div>
                </button>
              </div>
            </div>
          </Motion>
        </div>
      </div>
    </div>

    <!-- Full Screen Preview Modal -->
    <div v-if="selectedCamera"
      class="fixed inset-0 bg-black/80 backdrop-blur-xl z-50 flex items-center justify-center p-6"
      @click="closePreview">
      <div class="relative max-w-6xl max-h-full w-full h-full flex items-center justify-center">

        <!-- Close Button -->
        <button @click="closePreview"
          class="absolute top-6 right-6 z-20 backdrop-blur-md bg-black/30 hover:bg-black/50 text-white rounded-full p-3 transition-all duration-300 hover:scale-110 border border-white/20">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
          </svg>
        </button>

        <!-- Camera Label -->
        <div class="absolute top-6 left-6 z-20">
          <div class="backdrop-blur-md bg-black/30 border border-white/20 text-white px-4 py-2 rounded-xl">
            <span class="text-lg font-semibold">{{ selectedCamera.label || `Camera #${selectedCamera.id}` }}</span>
          </div>
        </div>

        <!-- Canvas Preview -->
        <canvas v-if="selectedCamera.active && selectedCamera.stream && renderMode === 'canvas'"
          :ref="(el) => previewCanvasEl = el as HTMLCanvasElement"
          class="max-w-full max-h-full object-contain rounded-2xl shadow-2xl border border-white/20"
          @click.stop></canvas>

        <!-- Video Preview -->
        <video v-if="selectedCamera.active && selectedCamera.stream && renderMode === 'video'"
          :ref="(el) => previewVideoEl = el as HTMLVideoElement"
          class="max-w-full max-h-full object-contain rounded-2xl shadow-2xl border border-white/20" autoplay muted
          playsinline @click.stop></video>

        <!-- No Stream Message -->
        <div v-else class="text-center text-white/70">
          <div class="backdrop-blur-md bg-white/10 rounded-2xl p-12 border border-white/20">
            <svg class="w-20 h-20 mx-auto mb-6 opacity-40" fill="currentColor" viewBox="0 0 20 20">
              <path d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z">
              </path>
            </svg>
            <p class="text-xl font-light">Camera is offline or no video stream available</p>
          </div>
        </div>
      </div>
    </div>
  </Motion>
</template>

<script setup lang="ts">
import { ref, reactive, onUnmounted, nextTick, onMounted } from 'vue'
import { Motion } from 'motion-v'
import { CameraController, formatCameraId } from '@/api/websocket/camera'
import type { DetectionResult } from '@/api/websocket/ws_detection'
import { getWarnings } from '@/api/inspection/API'
import type { Warning } from '@/types/apis/inspection_T'

interface Camera {
  id: number
  active: boolean
  stream: MediaStream | null
  deviceId?: string
  label?: string
  detectorId?: string
}

interface Stats {
  onlinePeople: number
  withHelmet: number
  withoutHelmet: number
  complianceRate: number
}

const permissionRequested = ref(false)
const allCamerasActive = ref(false)
const availableDevices = ref<MediaDeviceInfo[]>([])
const cameras = ref<Camera[]>([])
const selectedCamera = ref<Camera | null>(null)
const renderMode = ref<'canvas' | 'video'>('canvas')
const canvasRefs = ref<Map<number, HTMLCanvasElement>>(new Map())
const videoRefs = ref<Map<number, HTMLVideoElement>>(new Map())
const videoElements = ref<Map<number, HTMLVideoElement>>(new Map())
const controllers = ref<Map<number, CameraController>>(new Map())
const detectionResults = ref<Map<number, DetectionResult>>(new Map())
const lastDetectionResults = ref<Map<number, DetectionResult>>(new Map())

// Animation variants
const pageVariants = {
  initial: { opacity: 0, y: 40 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.8, ease: 'easeOut' }
}

const cardVariants = {
  initial: { opacity: 0, scale: 0.95, y: 20 },
  animate: { opacity: 1, scale: 1, y: 0 },
  transition: { duration: 0.6, ease: 'easeOut' }
}

// Warning data
const warnings = ref<Warning[]>([])

// Real-time statistics
const stats = ref<Stats>({
  onlinePeople: 0,
  withHelmet: 0,
  withoutHelmet: 0,
  complianceRate: 0
})

// Update statistics
const updateStats = () => {
  let totalPeople = 0
  let withHelmet = 0
  let withoutHelmet = 0

  lastDetectionResults.value.forEach((result) => {
    if (result && result.detections) {
      result.detections.forEach((detection) => {
        totalPeople++
        if (detection.detection_type === 'wearing_hat') {
          withHelmet++
        } else if (detection.detection_type === 'no_hat') {
          withoutHelmet++
        }
      })
    }
  })

  stats.value.onlinePeople = totalPeople
  stats.value.withHelmet = withHelmet
  stats.value.withoutHelmet = withoutHelmet
  stats.value.complianceRate = totalPeople > 0 ? Math.round((withHelmet / totalPeople) * 100) : 0
}

let warningInterval: number | undefined;

const fetchWarnings = async () => {
  try {
    const res = await getWarnings({ page: 1, page_size: 20 })
    console.log('API Response:', res)
    if (res.data && res.data.warnings) {
      warnings.value = res.data.warnings
      console.log('Fetched warning data:', warnings.value)
    }
  } catch (error) {
    console.error('Failed to fetch warning data:', error)
  }
}

const getWarningLevelInfo = (level: 'info' | 'warning' | 'critical') => {
  switch (level) {
    case 'critical':
      return {
        text: 'Critical Alert',
        cardClass: 'bg-red-500/10 border-red-400/30 hover:bg-red-500/20',
        badgeClass: 'bg-red-500/30 text-red-200 border border-red-400/40',
        accentClass: 'bg-gradient-to-b from-red-400 to-red-600'
      }
    case 'warning':
      return {
        text: 'Warning',
        cardClass: 'bg-amber-500/10 border-amber-400/30 hover:bg-amber-500/20',
        badgeClass: 'bg-amber-500/30 text-amber-200 border border-amber-400/40',
        accentClass: 'bg-gradient-to-b from-amber-400 to-amber-600'
      }
    default:
      return {
        text: 'Info',
        cardClass: 'bg-blue-500/10 border-blue-400/30 hover:bg-blue-500/20',
        badgeClass: 'bg-blue-500/30 text-blue-200 border border-blue-400/40',
        accentClass: 'bg-gradient-to-b from-blue-400 to-blue-600'
      }
  }
}

const formatDateTime = (dateString: string) => {
  const date = new Date(dateString)
  return date.toLocaleString('en-US', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

onMounted(() => {
  fetchWarnings()
  warningInterval = window.setInterval(fetchWarnings, 60000) // Refresh every minute
})

onUnmounted(() => {
  if (warningInterval) {
    clearInterval(warningInterval)
  }
})

// Preview elements
const previewCanvasEl = ref<HTMLCanvasElement | null>(null)
const previewVideoEl = ref<HTMLVideoElement | null>(null)

// Set canvas element reference
const setCanvasRef = async (cameraId: number, el: HTMLCanvasElement | null) => {
  if (!el) return
  canvasRefs.value.set(cameraId, el)
  const camera = cameras.value.find(c => c.id === cameraId)
  if (camera?.stream) {
    const video = document.createElement('video')
    video.srcObject = camera.stream
    video.muted = true
    video.autoplay = true
    video.playsInline = true
    videoElements.value.set(cameraId, video)

    video.addEventListener('loadedmetadata', () => {
      drawVideoToCanvas(video, el, cameraId)
    })

    try {
      await video.play()
    } catch (error) {
      console.warn(`Video playback failed (camera ${cameraId}):`, error)
    }
  }
}

// Set video element reference
const setVideoRef = (cameraId: number, el: HTMLVideoElement | null) => {
  if (!el) return
  videoRefs.value.set(cameraId, el)
  const camera = cameras.value.find(c => c.id === cameraId)
  if (camera?.stream) {
    el.srcObject = camera.stream
    el.play().catch(error => {
      console.warn(`Video playback failed (camera ${cameraId}):`, error)
    })
  }
}

// Draw video frame to canvas
const drawVideoToCanvas = (video: HTMLVideoElement, canvas: HTMLCanvasElement, camId?: number) => {
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  const draw = () => {
    if (video.readyState >= 2) {
      const rect = canvas.getBoundingClientRect()
      canvas.width = rect.width
      canvas.height = rect.height

      const videoAspect = video.videoWidth / video.videoHeight
      const canvasAspect = canvas.width / canvas.height

      let drawWidth: number, drawHeight: number, drawX: number, drawY: number

      if (videoAspect > canvasAspect) {
        drawWidth = canvas.width
        drawHeight = canvas.width / videoAspect
        drawX = 0
        drawY = (canvas.height - drawHeight) / 2
      } else {
        drawHeight = canvas.height
        drawWidth = canvas.height * videoAspect
        drawX = (canvas.width - drawWidth) / 2
        drawY = 0
      }

      ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.drawImage(video, drawX, drawY, drawWidth, drawHeight)

      if (renderMode.value === 'canvas' && camId != null) {
        const result = detectionResults.value.get(camId)
        if (result && result.detections && result.detections.length) {
          const scaleX = drawWidth / video.videoWidth
          const scaleY = drawHeight / video.videoHeight
          ctx.save()
          ctx.lineWidth = 2
          for (const det of result.detections) {
            const { x: bboxX, y: bboxY, width: bboxWidth, height: bboxHeight } = det.bbox
            const x = drawX + bboxX * scaleX
            const y = drawY + bboxY * scaleY
            const w = bboxWidth * scaleX
            const h = bboxHeight * scaleY
            ctx.strokeStyle = det.detection_type === 'wearing_hat' ? '#22c55e' : (det.detection_type === 'no_hat' ? '#ef4444' : '#f59e0b')
            ctx.beginPath()
            ctx.rect(x, y, w, h)
            ctx.stroke()
          }
          ctx.restore()
        }
      }
    }

    if (!video.paused && !video.ended) {
      requestAnimationFrame(draw)
    }
  }

  draw()
}

const openPreview = async (camera: Camera) => {
  if (camera.active && camera.stream) {
    selectedCamera.value = camera
    await nextTick()

    if (renderMode.value === 'canvas' && previewCanvasEl.value && camera.stream) {
      if (!previewVideoEl.value) {
        previewVideoEl.value = document.createElement('video')
      }
      previewVideoEl.value.srcObject = camera.stream
      previewVideoEl.value.muted = true
      previewVideoEl.value.autoplay = true
      previewVideoEl.value.playsInline = true

      previewVideoEl.value.addEventListener('loadedmetadata', () => {
        if (previewVideoEl.value && previewCanvasEl.value) {
          drawVideoToCanvas(previewVideoEl.value, previewCanvasEl.value, selectedCamera.value?.id)
        }
      })

      try {
        await previewVideoEl.value.play()
      } catch (error) {
        console.warn('Preview video playback failed:', error)
      }
    } else if (renderMode.value === 'video' && previewVideoEl.value && camera.stream) {
      previewVideoEl.value.srcObject = camera.stream
      try {
        await previewVideoEl.value.play()
      } catch (error) {
        console.warn('Preview video playback failed:', error)
      }
    }
  }
}

const closePreview = () => {
  if (previewVideoEl.value) {
    previewVideoEl.value.srcObject = null
    previewVideoEl.value.pause()
  }

  if (previewCanvasEl.value) {
    const ctx = previewCanvasEl.value.getContext('2d')
    if (ctx) {
      ctx.clearRect(0, 0, previewCanvasEl.value.width, previewCanvasEl.value.height)
    }
  }

  selectedCamera.value = null
}

// Get available cameras
const getAvailableCameras = async () => {
  const devices = await navigator.mediaDevices.enumerateDevices()
  availableDevices.value = devices.filter(d => d.kind === 'videoinput')
  cameras.value = availableDevices.value.map((d, i) => ({
    id: i + 1,
    active: false,
    stream: null,
    deviceId: d.deviceId,
    label: d.label || `Camera ${i + 1}`,
    detectorId: formatCameraId(i + 1)
  }))
}

// Request camera permission
const requestCameraPermission = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true })
    stream.getTracks().forEach(t => t.stop())
    permissionRequested.value = true
    await getAvailableCameras()
  } catch (e) {
    alert('Unable to access camera permissions')
  }
}

// Toggle individual camera
const toggleCamera = async (id: number) => {
  const camera = cameras.value.find(c => c.id === id)
  if (!camera) return

  if (camera.active) {
    camera.stream?.getTracks().forEach(t => t.stop())
    camera.stream = null
    camera.active = false

    const canvas = canvasRefs.value.get(id)
    if (canvas) {
      const ctx = canvas.getContext('2d')
      if (ctx) {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
      }
    }

    const hiddenVideo = videoElements.value.get(id)
    if (hiddenVideo) {
      hiddenVideo.srcObject = null
      hiddenVideo.pause()
      videoElements.value.delete(id)
    }

    const video = videoRefs.value.get(id)
    if (video) {
      video.srcObject = null
      video.pause()
    }

    const controller = controllers.value.get(id)
    if (controller) {
      controller.stopDetection()
      controller.disconnect()
      controllers.value.delete(id)
    }

    detectionResults.value.delete(id)
    lastDetectionResults.value.delete(id)
    updateStats()
  } else {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { deviceId: camera.deviceId ? { exact: camera.deviceId } : undefined }
      })
      camera.stream = stream
      camera.active = true
      await nextTick()

      if (renderMode.value === 'canvas') {
        const canvas = canvasRefs.value.get(id)
        if (canvas) {
          const video = document.createElement('video')
          video.srcObject = stream
          video.muted = true
          video.autoplay = true
          video.playsInline = true
          videoElements.value.set(id, video)

          video.addEventListener('loadedmetadata', () => {
            drawVideoToCanvas(video, canvas, id)
            let controller = controllers.value.get(id)
            if (!controller) {
              controller = new CameraController(camera.detectorId || formatCameraId(id), (result) => {
                detectionResults.value.set(id, result)
                lastDetectionResults.value.set(id, result)
                updateStats()
              })
              controllers.value.set(id, controller)
            }
            controller.setSource(video)
            controller.startDetection()
          })

          try {
            await video.play()
          } catch (playError) {
            console.warn('Video autoplay failed, user interaction may be required:', playError)
          }
        }
      } else {
        const video = videoRefs.value.get(id)
        if (video) {
          video.srcObject = stream
          video.addEventListener('loadedmetadata', () => {
            let controller = controllers.value.get(id)
            if (!controller) {
              controller = new CameraController(camera.detectorId || formatCameraId(id), (result) => {
                detectionResults.value.set(id, result)
                lastDetectionResults.value.set(id, result)
                updateStats()
              })
              controllers.value.set(id, controller)
            }
            controller.setSource(video)
            controller.startDetection()
          })
          try {
            await video.play()
          } catch (playError) {
            console.warn('Video autoplay failed, user interaction may be required:', playError)
          }
        }
      }
    } catch (e) {
      alert(`Camera ${id} startup failed`)
    }
  }
  allCamerasActive.value = cameras.value.every(c => c.active)
}

// Toggle all cameras
const toggleAllCameras = () => {
  const should = !allCamerasActive.value
  cameras.value.forEach(c => {
    if (c.active !== should) toggleCamera(c.id)
  })
}

// Export warnings as CSV
const exportWarnings = () => {
  if (warnings.value.length === 0) {
    alert('No warning records to export')
    return
  }

  try {
    const headers = ['Warning Level', 'Camera', 'Title', 'Time']
    const csvContent = [
      headers.join(','),
      ...warnings.value.map(warning => {
        const level = warning.warning_level === 'critical' ? 'Critical Alert' :
          warning.warning_level === 'warning' ? 'Warning' : 'Info'
        const camera = warning.camera_name || ''
        const title = warning.title ? `"${warning.title.replace(/"/g, '""')}"` : ''
        const time = formatDateTime(warning.created_at)
        return [level, camera, title, time].join(',')
      })
    ].join('\n')

    const BOM = '\uFEFF'
    const blob = new Blob([BOM + csvContent], { type: 'text/csv;charset=utf-8' })

    const url = window.URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.download = `alert_report_${new Date().toISOString().split('T')[0]}.csv`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    window.URL.revokeObjectURL(url)
  } catch (error) {
    console.error('Export failed:', error)
    alert('Export failed, please try again')
  }
}

// Reset warnings
const resetWarnings = () => {
  if (confirm('Are you sure you want to clear all alert records?')) {
    warnings.value = []
  }
}

// Toggle render mode
const toggleRenderMode = async () => {
  const newMode = renderMode.value === 'canvas' ? 'video' : 'canvas'

  const activeCameras = cameras.value.filter(c => c.active).map(c => c.id)

  for (const cameraId of activeCameras) {
    await toggleCamera(cameraId)
  }

  if (renderMode.value === 'canvas') {
    canvasRefs.value.forEach(canvas => {
      const ctx = canvas.getContext('2d')
      if (ctx) {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
      }
    })
    canvasRefs.value.clear()

    videoElements.value.forEach(video => {
      video.srcObject = null
      video.pause()
    })
    videoElements.value.clear()
  } else {
    videoRefs.value.forEach(video => {
      video.srcObject = null
      video.pause()
    })
    videoRefs.value.clear()
  }

  if (selectedCamera.value) {
    closePreview()
  }

  renderMode.value = newMode

  await nextTick()

  for (const cameraId of activeCameras) {
    await toggleCamera(cameraId)
  }
}

// Cleanup
onUnmounted(() => {
  cameras.value.forEach(c => c.stream?.getTracks().forEach(t => t.stop()))

  videoElements.value.forEach(video => {
    video.srcObject = null
    video.pause()
  })
  videoElements.value.clear()

  videoRefs.value.forEach(video => {
    video.srcObject = null
    video.pause()
  })
  videoRefs.value.clear()

  controllers.value.forEach(c => c.disconnect())
  controllers.value.clear()

  if (previewVideoEl.value) {
    previewVideoEl.value.srcObject = null
    previewVideoEl.value.pause()
  }

  canvasRefs.value.forEach(canvas => {
    const ctx = canvas.getContext('2d')
    if (ctx) {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
    }
  })
  canvasRefs.value.clear()
})
</script>

<style scoped>
/* Custom scrollbar for glassmorphism theme */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
  backdrop-filter: blur(10px);
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
  backdrop-filter: blur(10px);
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* Enhance glassmorphism effect */
@supports (backdrop-filter: blur()) {
  .backdrop-blur-xl {
    backdrop-filter: blur(20px);
  }

  .backdrop-blur-md {
    backdrop-filter: blur(12px);
  }
}

/* Smooth animations */
* {
  transition-property: all;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Gradient text enhancement */
.bg-gradient-to-r {
  background-attachment: fixed;
}
</style>
