<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import AvatarOwn from './AvatarOwn.vue'

// Reactive window dimensions for responsive behavior
const windowWidth = ref(window.innerWidth)
const windowHeight = ref(window.innerHeight)

// Computed properties for responsive sizing
const isMobile = computed(() => windowWidth.value <= 768)
const isTablet = computed(() => windowWidth.value > 768 && windowWidth.value <= 1024)
const isLandscape = computed(() => windowWidth.value > windowHeight.value)

// Handle window resize
function handleResize() {
  windowWidth.value = window.innerWidth
  windowHeight.value = window.innerHeight
}

onMounted(() => {
  window.addEventListener('resize', handleResize)

  handleResize()
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})

// Reactive 5-byte packed data for the dnaBytes prop
const dnaBytes = ref<number[]>([0, 0, 0, 0, 0])

// --- Avataaars Style Options ---
// Simplified options based on DiceBear avataaars components
const skinColors = ['#fdbcb4', '#f1c27d', '#e0ac69', '#c68642', '#8d5524']
const hairColors = ['#2c1b18', '#4a312c', '#6b4423', '#8b5a3c', '#a0774d', '#b6915f', '#c9a875', '#d9c4a7']
const clothingColors = ['#ff6b6b', '#4ecdc4', '#45b7d1', '#96ceb4', '#ffeaa7', '#dda0dd', '#98d8c8', '#f7dc6f']
const backgroundColors = [
  'linear-gradient(135deg, #FFB3BA 0%, #FFDFBA 100%)',
  'linear-gradient(135deg, #BAE1FF 0%, #D1BAFF 100%)',
  'linear-gradient(135deg, #BAFFC9 0%, #BAF7FF 100%)',
  'linear-gradient(135deg, #D1BAFF 0%, #E8BAFF 100%)',
  'linear-gradient(135deg, #FFD1BA 0%, #FFF8BA 100%)',
  'linear-gradient(135deg, #BAF7FF 0%, #D1FFBA 100%)',
  'linear-gradient(135deg, #E8BAFF 0%, #FFB3BA 100%)',
  'linear-gradient(135deg, #FFF8BA 0%, #BAFFC9 100%)',
  'linear-gradient(135deg, #FFBABA 0%, #FFD1BA 100%)',
  'linear-gradient(135deg, #D1FFBA 0%, #BAE1FF 100%)'
]
const backgroundNames = ['Peach Sunset', 'Purple Ocean', 'Mint Breeze', 'Lavender Dream', 'Orange Cream', 'Aqua Sky', 'Pink Lavender', 'Lemon Lime', 'Coral Peach', 'Blue Mint']
const clothingStyles = ['Blazer & Shirt', 'Blazer & Sweater', 'Collar Sweater', 'Graphic T-Shirt', 'Hoodie', 'Overall', 'Crew Neck Shirt', 'Scoop Neck Shirt', 'V-Neck Shirt']
const hairStyles = ['Bald', 'Bob Cut', 'Bun', 'Curly Hair', 'Curvy', 'Dreads', 'Dreads 02', 'Frida', 'Frizzle', 'Fro Band', 'Long Hair', 'Mia Wallace', 'Shaggy Mullet', 'Shaved Sides', 'Short Curly', 'Short Flat', 'Short Round', 'Short Waved', 'Sides', 'Straight 01', 'Straight 02', 'Straight and Strand', 'Caesar Cut', 'Caesar and Side Part', 'Big Hair']
const eyeTypes = ['Normal', 'Closed', 'Happy', 'Winking']
const mouthTypes = ['Neutral', 'Smiling', 'Concerned', 'Grimacing']
const facialHairTypes = ['None', 'Beard Light', 'Beard Medium', 'Mustache Fancy', 'Mustache Magnum']
const accessoryTypes = ['None', 'Eyepatch', 'Kurt', 'Prescription 01', 'Prescription 02', 'Round', 'Sunglasses', 'Wayfarers']

// Helper function to create computed property for packed DNA bits
function createPackedComputed(startBit: number, bits: number) {
  const byteIndex = Math.floor(startBit / 8)
  const bitOffset = startBit % 8
  return computed({
    get: () => {
      let value = (dnaBytes.value[byteIndex]! >> bitOffset) & ((1 << Math.min(bits, 8 - bitOffset)) - 1)
      if (bits > (8 - bitOffset)) {
        const remainingBits = bits - (8 - bitOffset)
        value |= (dnaBytes.value[byteIndex + 1]! & ((1 << remainingBits) - 1)) << (8 - bitOffset)
      }
      return value
    },
    set: (val: number) => {
      const firstBits = Math.min(bits, 8 - bitOffset)
      const secondBits = bits - firstBits
      dnaBytes.value[byteIndex] = (dnaBytes.value[byteIndex]! & ~(((1 << firstBits) - 1) << bitOffset)) | ((val & ((1 << firstBits) - 1)) << bitOffset)
      dnaBytes.value[byteIndex] &= 0xFF
      if (secondBits > 0 && byteIndex + 1 < dnaBytes.value.length) {
        const nextByteIndex = byteIndex + 1
        const nextByte = dnaBytes.value[nextByteIndex]!
        dnaBytes.value[nextByteIndex] = (nextByte & ~((1 << secondBits) - 1)) | ((val >> firstBits) & ((1 << secondBits) - 1))
        dnaBytes.value[nextByteIndex] &= 0xFF
      }
    },
  })
}

// Create computed properties for each part of the DNA
const version = createPackedComputed(0, 3) // 3 bits for version (0-7) - FIRST
const skinColor = createPackedComputed(3, 3) // 3 bits
const hairStyle = createPackedComputed(6, 5) // 5 bits
const eyes = createPackedComputed(11, 4) // 4 bits (expanded for future)
const mouth = createPackedComputed(15, 4) // 4 bits (expanded for future)
const facialHair = createPackedComputed(19, 4) // 4 bits (expanded for future)
const hairColor = createPackedComputed(23, 3) // 3 bits
const clothingColor = createPackedComputed(26, 3) // 3 bits
const clothingStyle = createPackedComputed(29, 4) // 4 bits
const backgroundColor = createPackedComputed(33, 4) // 4 bits
const accessories = createPackedComputed(37, 3) // 3 bits for accessories (0-7)

// Set initial DNA values
skinColor.value = 1
hairStyle.value = 0
eyes.value = 2
mouth.value = 0
facialHair.value = 0
hairColor.value = 0
clothingColor.value = 2 // Blue color to make Graphic T-Shirt more visible
clothingStyle.value = 3 // Graphic T-Shirt
backgroundColor.value = 0
version.value = 1 // Current format version
accessories.value = 0 // None

// Computed property for base64 encoded DNA
const dnaBase64 = computed(() => {
  const bytes = new Uint8Array(dnaBytes.value)
  return btoa(String.fromCharCode(...bytes))
})

// Computed property for DNA length
const dnaLength = computed(() => dnaBytes.value.length)

// Computed property for DNA breakdown
const dnaBreakdown = computed(() => [
  { label: 'Skin Color', value: skinColor.value },
  { label: 'Hair Style', value: hairStyle.value },
  { label: 'Eye Type', value: eyes.value },
  { label: 'Mouth Type', value: mouth.value },
  { label: 'Facial Hair', value: facialHair.value },
  { label: 'Hair Color', value: hairColor.value },
  { label: 'Clothing Color', value: clothingColor.value },
  { label: 'Clothing Style', value: clothingStyle.value },
  { label: 'Background', value: backgroundColor.value },
  { label: 'Accessories', value: accessories.value },
  { label: 'Version', value: version.value },
])

// Ref for loading DNA from base64
const loadDnaBase64 = ref('')

function loadFromBase64() {
  try {
    const decoded = atob(loadDnaBase64.value)
    const bytes = new Uint8Array(decoded.length)
    for (let i = 0; i < decoded.length; i++) {
      bytes[i] = decoded.charCodeAt(i)
    }
    if (bytes.length !== 5) {
      console.error('Invalid DNA: must be exactly 5 bytes')
      return
    }
    dnaBytes.value = Array.from(bytes)
    // Check version
    if (version.value !== 1) {
      console.error('Unsupported DNA version:', version.value)
      // Reset to default
      skinColor.value = 1
      hairStyle.value = 0
      eyes.value = 2
      mouth.value = 0
      facialHair.value = 0
      hairColor.value = 0
      clothingColor.value = 0
      clothingStyle.value = 0
      backgroundColor.value = 0
      version.value = 1
      return
    }
    loadDnaBase64.value = ''
  } catch (error) {
    console.error('Invalid base64 DNA:', error)
  }
}

function randomize() {
  skinColor.value = Math.floor(Math.random() * skinColors.length)
  hairStyle.value = Math.floor(Math.random() * hairStyles.length)
  eyes.value = Math.floor(Math.random() * eyeTypes.length)
  mouth.value = Math.floor(Math.random() * mouthTypes.length)
  facialHair.value = Math.floor(Math.random() * facialHairTypes.length)
  hairColor.value = Math.floor(Math.random() * hairColors.length)
  clothingColor.value = Math.floor(Math.random() * clothingColors.length)
  clothingStyle.value = Math.floor(Math.random() * clothingStyles.length)
  backgroundColor.value = Math.floor(Math.random() * backgroundColors.length)
  accessories.value = Math.floor(Math.random() * accessoryTypes.length)
  version.value = 1 // Always set to current version
  dnaBytes.value = [...dnaBytes.value] // trigger reactivity
}
</script>

<template>
  <div class="app-layout" :class="{ 
    'mobile-layout': isMobile, 
    'tablet-layout': isTablet, 
    'landscape-layout': isLandscape && windowHeight < 600 
  }">
    <div class="main-content">
      <div class="avatar-section">
        <div class="avatars-display">
          <div class="avatar-container main-avatar" :style="{ background: backgroundColors[backgroundColor] }">
            <AvatarOwn 
              :key="dnaBytes.join(',')" 
              :dnaBytes="dnaBytes" 
              :size="isMobile ? Math.min(180, windowWidth * 0.4) : 200" 
            />
          </div>
          <div class="smaller-avatars">
            <div class="avatar-container medium-avatar" :style="{ background: backgroundColors[backgroundColor] }">
              <AvatarOwn 
                :key="dnaBytes.join(',')" 
                :dnaBytes="dnaBytes" 
                :size="64" 
              />
              <span class="medium-label">Profile Size</span>
            </div>
            <div class="avatar-container mobile-avatar" :style="{ background: backgroundColors[backgroundColor] }">
              <AvatarOwn 
                :key="dnaBytes.join(',')" 
                :dnaBytes="dnaBytes" 
                :size="32" 
              />
              <span class="mobile-label">Mobile Size</span>
            </div>
          </div>
        </div>
        <button @click="randomize" class="randomize-btn">Randomize</button>
        <div class="dna-section">
          <h4>DNA Breakdown</h4>
          <div class="dna-info">Length: {{ dnaLength }} bytes</div>
          <div class="dna-breakdown">
            <div v-for="(item, index) in dnaBreakdown" :key="index" class="dna-item">
              <span class="dna-label">{{ item.label }}:</span>
              <span class="dna-value">{{ item.value }}</span>
            </div>
          </div>
          <div class="dna-bytes">Bytes: [{{ dnaBytes.map(b => b & 0xFF).join(', ') }}]</div>
          <h4>Base64</h4>
          <input type="text" :value="dnaBase64" readonly @click="(e) => (e.target as HTMLInputElement).select()" class="dna-code" />
          <div class="dna-load">
            <input type="text" v-model="loadDnaBase64" placeholder="Paste code here" />
            <button @click="loadFromBase64">Load</button>
          </div>
        </div>
      </div>

      <div class="controls-panel">
        <div class="colors-section">
          <h3>Colours</h3>
          <div class="color-group">
            <div class="color-label">Skin:</div>
            <div class="color-circles">
              <div
                v-for="(color, i) in skinColors"
                :key="i"
                class="color-circle"
                :class="{ selected: skinColor === i }"
                :style="{ backgroundColor: color }"
                @click="skinColor = i"
                :title="color"
              ></div>
            </div>
          </div>
          <div class="color-group">
            <div class="color-label">Hair:</div>
            <div class="color-circles">
              <div
                v-for="(color, i) in hairColors"
                :key="i"
                class="color-circle"
                :class="{ selected: hairColor === i }"
                :style="{ backgroundColor: color }"
                @click="hairColor = i"
                :title="color"
              ></div>
            </div>
          </div>
          <div class="color-group">
            <div class="color-label">Clothing:</div>
            <div class="color-circles">
              <div
                v-for="(color, i) in clothingColors"
                :key="i"
                class="color-circle"
                :class="{ selected: clothingColor === i }"
                :style="{ backgroundColor: color }"
                @click="clothingColor = i"
                :title="color"
              ></div>
            </div>
          </div>
          <div class="color-group">
            <div class="color-label">Background:</div>
            <div class="color-circles">
              <div
                v-for="(color, i) in backgroundColors"
                :key="i"
                class="color-circle"
                :class="{ selected: backgroundColor === i }"
                :style="{ background: color }"
                @click="backgroundColor = i"
                :title="backgroundNames[i]"
              ></div>
            </div>
          </div>
        </div>

        <div class="options-section">
          <h3>Options</h3>
          <div class="control-group">
            <label>Eyes:
              <select v-model.number="eyes">
                <option v-for="(type, i) in eyeTypes" :key="i" :value="i">{{ type }}</option>
              </select>
            </label>
            <label>Mouth:
              <select v-model.number="mouth">
                <option v-for="(type, i) in mouthTypes" :key="i" :value="i">{{ type }}</option>
              </select>
            </label>
          </div>

          <div class="control-group">
            <label>Hair Style:
              <select v-model.number="hairStyle">
                <option v-for="(style, i) in hairStyles" :key="i" :value="i">{{ style }}</option>
              </select>
            </label>
            <label>Facial Hair:
              <select v-model.number="facialHair">
                <option v-for="(type, i) in facialHairTypes" :key="i" :value="i">{{ type }}</option>
              </select>
            </label>
          </div>

          <div class="control-group">
            <label>Clothing Style:
              <select v-model.number="clothingStyle">
                <option v-for="(style, i) in clothingStyles" :key="i" :value="i">{{ style }}</option>
              </select>
            </label>
            <label>Accessories:
              <select v-model.number="accessories">
                <option v-for="(acc, i) in accessoryTypes" :key="i" :value="i">{{ acc }}</option>
              </select>
            </label>
          </div>
        </div>
      </div>
    </div>
  </div>
  

</template>

<style scoped>
.avatars-display {
  display: flex;
  gap: 2rem;
  align-items: center;
  justify-content: center;
}

.smaller-avatars {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
}

.avatar-container {
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 24px rgba(0,0,0,0.2);
  border: 4px solid #fff;
}

.main-avatar {
  width: min(220px, 40vw);
  height: min(220px, 40vw);
}

.medium-avatar {
  width: 80px;
  height: 80px;
  position: relative;
}

.mobile-avatar {
  width: 40px;
  height: 40px;
  position: relative;
}

.mobile-label,
.medium-label {
  position: absolute;
  bottom: -20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.75rem;
  color: #666;
  white-space: nowrap;
  text-align: center;
}



@media (max-width: 768px) {
  .avatars-display {
    flex-direction: column;
    gap: 1rem;
  }
  
  .mobile-avatar {
    width: 36px;
    height: 36px;
  }
  
  .license-notice {
    font-size: 0.6rem;
    bottom: 5px;
    right: 5px;
    padding: 3px 6px;
  }
}
</style>
