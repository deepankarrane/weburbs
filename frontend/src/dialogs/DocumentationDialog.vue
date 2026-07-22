<template>
  <Dialog
    v-model:visible="visible"
    modal
    header="WebURBS Documentation"
    :style="{ width: '90vw', maxWidth: '1200px' }"
    :maximizable="true"
    :closable="true"
  >
    <div class="documentation-content">
      <!-- Navigation Tabs -->
      <TabView v-model:activeIndex="activeTabIndex">
        <TabPanel header="Architecture" value="architecture">
          <div class="tab-content">
            <h3 class="text-xl font-bold mb-4 text-primary">System Architecture</h3>
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <div class="architecture-section">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Frontend Architecture</h4>
                <ul class="list-disc list-inside space-y-2 text-gray-700">
                  <li><strong>Framework:</strong> Vue.js 3 with TypeScript</li>
                  <li><strong>UI Library:</strong> PrimeVue components</li>
                  <li><strong>Styling:</strong> Tailwind CSS</li>
                  <li><strong>Build Tool:</strong> Vite</li>
                  <li><strong>State Management:</strong> Vue Composition API</li>
                  <li><strong>Routing:</strong> Vue Router</li>
                </ul>
              </div>
              
              <div class="architecture-section">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Backend Architecture</h4>
                <ul class="list-disc list-inside space-y-2 text-gray-700">
                  <li><strong>Framework:</strong> Django (Python)</li>
                  <li><strong>Database:</strong> PostgreSQL (default) / SQLite</li>
                  <li><strong>API:</strong> RESTful API with Django REST Framework</li>
                  <li><strong>Authentication:</strong> Django built-in auth system</li>
                  <li><strong>Optimization:</strong> URBS optimizer integration</li>
                </ul>
              </div>
            </div>
            
            <div class="mt-6">
              <h4 class="text-lg font-semibold mb-3 text-secondary">System Components</h4>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                  <div class="text-center p-3 bg-white rounded border">
                    <i class="pi pi-desktop text-2xl text-primary mb-2"></i>
                    <h5 class="font-semibold">Frontend</h5>
                    <p class="text-sm text-gray-600">User interface for configuration and visualization</p>
                  </div>
                  <div class="text-center p-3 bg-white rounded border">
                    <i class="pi pi-server text-2xl text-primary mb-2"></i>
                    <h5 class="font-semibold">Backend</h5>
                    <p class="text-sm text-gray-600">Data management and optimization engine</p>
                  </div>
                  <div class="text-center p-3 bg-white rounded border">
                    <i class="pi pi-database text-2xl text-primary mb-2"></i>
                    <h5 class="font-semibold">Database</h5>
                    <p class="text-sm text-gray-600">Persistent storage for projects and configurations</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </TabPanel>
        
        <TabPanel header="Modeling Steps" value="modeling">
          <div class="tab-content">
            <h3 class="text-xl font-bold mb-4 text-primary">Step-by-Step Modeling Guide</h3>
            
            <div class="space-y-6">
              <div class="step-item">
                <div class="step-number">1</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Project Setup</h4>
                  <p class="text-gray-700 mb-2">Create a new project and configure global settings including project name, description, and basic parameters.</p>
                  <div class="bg-blue-50 p-3 rounded border-l-4 border-blue-400">
                    <strong>Tip:</strong> Start with a descriptive project name and clear description to keep track of your work.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">2</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Site Configuration</h4>
                  <p class="text-gray-700 mb-2">Define geographical locations where your energy system components will be situated. Set coordinates and site-specific parameters.</p>
                  <div class="bg-green-50 p-3 rounded border-l-4 border-green-400">
                    <strong>Note:</strong> Sites can represent cities, regions, or specific locations depending on your modeling scope.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">3</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Commodity Definition</h4>
                  <p class="text-gray-700 mb-2">Define energy carriers and materials that flow through your system (electricity, heat, hydrogen, etc.).</p>
                  <div class="bg-yellow-50 p-3 rounded border-l-4 border-yellow-400">
                    <strong>Important:</strong> Set appropriate units and constraints for each commodity type.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">4</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Process Configuration</h4>
                  <p class="text-gray-700 mb-2">Define energy conversion and production processes (solar panels, wind turbines, power plants, etc.).</p>
                  <div class="bg-purple-50 p-3 rounded border-l-4 border-purple-400">
                    <strong>Key:</strong> Configure input/output commodities, efficiency, costs, and capacity constraints.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">5</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Demand Specification</h4>
                  <p class="text-gray-700 mb-2">Define energy demand patterns over time (8760 hours per year) for different commodities at each site.</p>
                  <div class="bg-orange-50 p-3 rounded border-l-4 border-orange-400">
                    <strong>Data:</strong> Use historical data or create synthetic demand profiles based on your use case.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">6</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Storage Systems</h4>
                  <p class="text-gray-700 mb-2">Configure energy storage solutions (batteries, thermal storage, etc.) with efficiency and cost parameters.</p>
                  <div class="bg-indigo-50 p-3 rounded border-l-4 border-indigo-400">
                    <strong>Consider:</strong> Storage capacity, charge/discharge efficiency, and degradation over time.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">7</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Transmission & Distribution</h4>
                  <p class="text-gray-700 mb-2">Set up energy transmission networks between sites with capacity and efficiency constraints.</p>
                  <div class="bg-red-50 p-3 rounded border-l-4 border-red-400">
                    <strong>Planning:</strong> Consider grid infrastructure costs and transmission losses.
                  </div>
                </div>
              </div>
              
              <div class="step-item">
                <div class="step-number">8</div>
                <div class="step-content">
                  <h4 class="text-lg font-semibold text-secondary">Simulation & Optimization</h4>
                  <p class="text-gray-700 mb-2">Run the URBS optimization to find the least-cost energy system configuration.</p>
                  <div class="bg-teal-50 p-3 rounded border-l-4 border-teal-400">
                    <strong>Analysis:</strong> Review results, costs, and system performance metrics.
                  </div>
                </div>
              </div>
            </div>
          </div>
        </TabPanel>
        
        <TabPanel header="Features" value="features">
          <div class="tab-content">
            <h3 class="text-xl font-bold mb-4 text-primary">Tool Features & Capabilities</h3>
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <div class="feature-category">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Core Modeling Features</h4>
                <div class="space-y-3">
                  <div class="feature-item">
                    <i class="pi pi-sitemap text-primary mr-2"></i>
                    <span><strong>Multi-site modeling:</strong> Configure energy systems across multiple geographical locations</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-bolt text-primary mr-2"></i>
                    <span><strong>Commodity management:</strong> Define various energy carriers and materials</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-hammer text-primary mr-2"></i>
                    <span><strong>Process configuration:</strong> Set up energy conversion technologies</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-gauge text-primary mr-2"></i>
                    <span><strong>Demand modeling:</strong> Create time-series demand profiles</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-warehouse text-primary mr-2"></i>
                    <span><strong>Storage systems:</strong> Configure energy storage solutions</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-wifi text-primary mr-2"></i>
                    <span><strong>Transmission networks:</strong> Model energy distribution systems</span>
                  </div>
                </div>
              </div>
              
              <div class="feature-category">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Advanced Features</h4>
                <div class="space-y-3">
                  <div class="feature-item">
                    <i class="pi pi-sun text-primary mr-2"></i>
                    <span><strong>SupIm integration:</strong> Automatic solar and wind data from Renewable Ninja</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-sliders-v text-primary mr-2"></i>
                    <span><strong>DSM (Demand Side Management):</strong> Load shifting and demand response</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-dollar text-primary mr-2"></i>
                    <span><strong>Buy/Sell pricing:</strong> Market interaction and price signals</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-hourglass text-primary mr-2"></i>
                    <span><strong>Time-varying efficiency:</strong> Seasonal and temporal efficiency changes</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-chart-line text-primary mr-2"></i>
                    <span><strong>Cost optimization:</strong> Minimize total system costs</span>
                  </div>
                  <div class="feature-item">
                    <i class="pi pi-calendar text-primary mr-2"></i>
                    <span><strong>8760-hour modeling:</strong> Full year hourly resolution</span>
                  </div>
                </div>
              </div>
            </div>
            
            <div class="mt-6">
              <h4 class="text-lg font-semibold mb-3 text-secondary">User Experience Features</h4>
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div class="text-center p-4 bg-blue-50 rounded-lg border">
                  <i class="pi pi-upload text-2xl text-blue-600 mb-2"></i>
                  <h5 class="font-semibold text-blue-800">Data Import</h5>
                  <p class="text-sm text-blue-700">Upload Excel files and JSON configurations</p>
                </div>
                <div class="text-center p-4 bg-green-50 rounded-lg border">
                  <i class="pi pi-eye text-2xl text-green-600 mb-2"></i>
                  <h5 class="font-semibold text-green-800">Visualization</h5>
                  <p class="text-sm text-green-700">Interactive charts and energy flow diagrams</p>
                </div>
                <div class="text-center p-4 bg-purple-50 rounded-lg border">
                  <i class="pi pi-cog text-2xl text-purple-600 mb-2"></i>
                  <h5 class="font-semibold text-purple-800">Templates</h5>
                  <p class="text-sm text-purple-700">Pre-configured default data and examples</p>
                </div>
              </div>
            </div>
          </div>
        </TabPanel>
        
        <TabPanel header="Configuration" value="configuration">
          <div class="tab-content">
            <h3 class="text-xl font-bold mb-4 text-primary">Configuration Guide</h3>
            
            <div class="space-y-6">
              <div class="config-section">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Default Data Configuration</h4>
                <p class="text-gray-700 mb-3">WebURBS provides pre-configured default data that can be customized for your specific use case.</p>
                <div class="bg-gray-50 p-4 rounded-lg">
                  <h5 class="font-semibold mb-2">Configuration Files Location:</h5>
                  <code class="bg-gray-200 px-2 py-1 rounded text-sm">backend/config/</code>
                  <ul class="list-disc list-inside mt-2 space-y-1 text-sm">
                    <li><code>base_coms.json</code> - Default commodities</li>
                    <li><code>base_demand.json</code> - Default demand patterns</li>
                    <li><code>base_procs.json</code> - Default processes</li>
                    <li><code>base_stor.json</code> - Default storage systems</li>
                  </ul>
                </div>
              </div>
              
              <div class="config-section">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Environment Variables</h4>
                <div class="overflow-x-auto">
                  <table class="min-w-full border border-gray-300">
                    <thead class="bg-gray-100">
                      <tr>
                        <th class="border border-gray-300 px-3 py-2 text-left">Variable</th>
                        <th class="border border-gray-300 px-3 py-2 text-left">Description</th>
                        <th class="border border-gray-300 px-3 py-2 text-left">Example</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr>
                        <td class="border border-gray-300 px-3 py-2 font-mono text-sm">RN_KEY</td>
                        <td class="border border-gray-300 px-3 py-2">Renewable Ninja API key</td>
                        <td class="border border-gray-300 px-3 py-2 font-mono text-sm">your_api_key_here</td>
                      </tr>
                      <tr>
                        <td class="border border-gray-300 px-3 py-2 font-mono text-sm">SECRET_KEY</td>
                        <td class="border border-gray-300 px-3 py-2">Django secret key</td>
                        <td class="border border-gray-300 px-3 py-2 font-mono text-sm">change_me</td>
                      </tr>
                      <tr>
                        <td class="border border-gray-300 px-3 py-2 font-mono text-sm">VUE_APP_API_URL</td>
                        <td class="border border-gray-300 px-3 py-2">Frontend API endpoint</td>
                        <td class="border border-gray-300 px-3 py-2 font-mono text-sm">https://your.domain</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
              
              <div class="config-section">
                <h4 class="text-lg font-semibold mb-3 text-secondary">Data Upload Formats</h4>
                <div class="bg-yellow-50 p-4 rounded border-l-4 border-yellow-400">
                  <h5 class="font-semibold text-yellow-800 mb-2">Supported Formats:</h5>
                  <ul class="list-disc list-inside space-y-1 text-yellow-700">
                    <li><strong>Excel (.xlsx):</strong> Multi-sheet configuration files</li>
                    <li><strong>JSON:</strong> Structured configuration data</li>
                    <li><strong>Time Series:</strong> 8760-hour data arrays</li>
                  </ul>
                </div>
              </div>
            </div>
          </div>
        </TabPanel>
      </TabView>
    </div>
    
    <template #footer>
      <div class="flex justify-between items-center">
        <div class="text-sm text-gray-600">
          <span v-if="activeTabIndex === 0">Architecture Overview</span>
          <span v-else-if="activeTabIndex === 1">Modeling Steps</span>
          <span v-else-if="activeTabIndex === 2">Tool Features</span>
          <span v-else-if="activeTabIndex === 3">Configuration Guide</span>
        </div>
        <Button label="Close" icon="pi pi-times" @click="closeDialog" />
      </div>
    </template>
  </Dialog>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Dialog from 'primevue/dialog'
import TabView from 'primevue/tabview'
import TabPanel from 'primevue/tabpanel'
import Button from 'primevue/button'

const visible = defineModel<boolean>('visible', { default: false })

const activeTabIndex = ref(0)

function closeDialog() {
  visible.value = false
}
</script>

<style scoped>
.documentation-content {
  max-height: 70vh;
  overflow-y: auto;
}

.tab-content {
  padding: 1rem 0;
}

.step-item {
  display: flex;
  gap: 1rem;
  align-items: flex-start;
}

.step-number {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  width: 2rem;
  height: 2rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  flex-shrink: 0;
  margin-top: 0.25rem;
}

.step-content {
  flex: 1;
}

.feature-item {
  display: flex;
  align-items: center;
  padding: 0.5rem;
  background: #f8fafc;
  border-radius: 0.5rem;
  border-left: 3px solid #e2e8f0;
}

.feature-category {
  background: #f8fafc;
  padding: 1.5rem;
  border-radius: 0.75rem;
  border: 1px solid #e2e8f0;
}

.config-section {
  background: white;
  padding: 1.5rem;
  border-radius: 0.75rem;
  border: 1px solid #e2e8f0;
}

.architecture-section {
  background: #f8fafc;
  padding: 1.5rem;
  border-radius: 0.75rem;
  border: 1px solid #e2e8f0;
}

code {
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
}
</style>
