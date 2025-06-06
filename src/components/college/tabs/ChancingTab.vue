<template>
  <v-row>
    <v-col cols="12" md="8">
      <v-card flat class="mb-6" style="background-color: transparent;">
        <v-card-title class="text-h5 font-weight-bold pb-4">
          Detailed Admission Chances for {{ college.name }}
        </v-card-title>

        <!-- Controls Section -->
        <v-card class="mb-6 pa-4">
          <v-row>
            <v-col cols="12">
              <div class="d-flex align-center justify-space-around">
                <div class="d-flex align-center" v-if="college.allowsEarlyDecision">
                  <div class="me-3">
                    <div class="text-subtitle-1 font-weight-medium">Early Decision</div>
                    <div class="text-caption">Increases chances</div>
                  </div>
                  <v-switch
                    :model-value="isEarlyDecision"
                    @update:modelValue="$emit('update:isEarlyDecision', $event); $emit('calculateDetailedChance')"
                    color="primary"
                    hide-details
                    density="compact"
                    :disabled="!college.allowsEarlyDecision"
                  ></v-switch>
                </div>
                <div v-else class="d-flex align-center">
                  <div class="me-3">
                    <div class="text-subtitle-1 font-weight-medium">Early Decision</div>
                    <div class="text-caption text-disabled">Not offered</div>
                  </div>
                   <v-switch
                    :model-value="false"
                    color="primary"
                    hide-details
                    density="compact"
                    disabled
                  ></v-switch>
                </div>

                <v-divider vertical class="mx-4"></v-divider>

                <div class="d-flex align-center" v-if="college.considersLegacy">
                  <div class="me-3">
                    <div class="text-subtitle-1 font-weight-medium">Legacy Status</div>
                    <div class="text-caption">Family attended</div>
                  </div>
                  <v-switch
                    :model-value="isLegacy"
                    @update:modelValue="$emit('update:isLegacy', $event); $emit('calculateDetailedChance')"
                    color="primary"
                    hide-details
                    density="compact"
                    :disabled="!college.considersLegacy"
                  ></v-switch>
                </div>
                 <div v-else class="d-flex align-center">
                  <div class="me-3">
                    <div class="text-subtitle-1 font-weight-medium">Legacy Status</div>
                    <div class="text-caption text-disabled">Not considered</div>
                  </div>
                   <v-switch
                    :model-value="false"
                    color="primary"
                    hide-details
                    density="compact"
                    disabled
                  ></v-switch>
                </div>
              </div>
            </v-col>
          </v-row>
        </v-card>

        <!-- Chance Overview -->
        <v-card class="mb-6 pa-4">
          <v-row>
            <v-col cols="12" md="6">
              <div class="d-flex flex-column align-center justify-center">
                <v-progress-circular
                  :model-value="detailedCollegeChance.probabilityPercentage"
                  :color="detailedChanceColor"
                  :size="150"
                  :width="15"
                  class="mb-3"
                >
                  <span class="text-h4">{{ detailedCollegeChance.probabilityPercentage }}%</span>
                </v-progress-circular>
                <div class="text-h6 font-weight-bold">{{ detailedChanceDescription }}</div>
                <div class="text-body-2 text-center mt-2">
                  {{ detailedCollegeChance.timesAverageApplicant || '1.0' }} times the average applicant
                </div>
              </div>
            </v-col>

            <v-col cols="12" md="6">
              <v-list density="compact">
                <v-list-subheader class="text-subtitle-1 font-weight-medium">Your Profile Strengths</v-list-subheader>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon :color="getZScoreColor(detailedCollegeChance.zScores?.gpa || 0)" class="mr-2">
                      mdi-school
                    </v-icon>
                  </template>
                  <v-list-item-title>GPA</v-list-item-title>
                  <template v-slot:append>
                    <v-chip :color="getZScoreColor(detailedCollegeChance.zScores?.gpa || 0)" size="small">
                      {{ formatZScore(detailedCollegeChance.zScores?.gpa || 0) }}σ
                    </v-chip>
                  </template>
                </v-list-item>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon :color="getZScoreColor(detailedCollegeChance.zScores?.sat || 0)" class="mr-2">
                      mdi-book-open-variant
                    </v-icon>
                  </template>
                  <v-list-item-title>SAT</v-list-item-title>
                  <template v-slot:append>
                    <v-chip :color="getZScoreColor(detailedCollegeChance.zScores?.sat || 0)" size="small">
                      {{ formatZScore(detailedCollegeChance.zScores?.sat || 0) }}σ
                    </v-chip>
                  </template>
                </v-list-item>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon :color="getZScoreColor(detailedCollegeChance.zScores?.ap || 0)" class="mr-2">
                      mdi-certificate
                    </v-icon>
                  </template>
                  <v-list-item-title>AP Courses</v-list-item-title>
                  <template v-slot:append>
                    <v-chip :color="getZScoreColor(detailedCollegeChance.zScores?.ap || 0)" size="small">
                      {{ formatZScore(detailedCollegeChance.zScores?.ap || 0) }}σ
                    </v-chip>
                  </template>
                </v-list-item>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon :color="getZScoreColor(detailedCollegeChance.zScores?.ec || 0)" class="mr-2">
                      mdi-account-group
                    </v-icon>
                  </template>
                  <v-list-item-title>Extracurriculars</v-list-item-title>
                  <template v-slot:append>
                    <v-chip :color="getZScoreColor(detailedCollegeChance.zScores?.ec || 0)" size="small">
                      {{ formatZScore(detailedCollegeChance.zScores?.ec || 0) }}σ
                    </v-chip>
                  </template>
                </v-list-item>
              </v-list>
            </v-col>
          </v-row>
        </v-card>

        <!-- Recommendation -->
        <v-card class="mb-6 pa-4">
          <div class="d-flex align-center justify-space-between mb-3">
            <div class="text-subtitle-1 font-weight-medium">Admission Recommendation</div>
            <v-btn
              v-if="!loadingAiRec"
              color="primary"
              variant="tonal"
              size="small"
              @click="$emit('toggleAiRecommendation')"
            >
              <v-icon size="small" class="mr-1">mdi-robot</v-icon>
              {{ useAiRecommendation ? 'Using AI Insight' : 'Get AI Insight' }}
            </v-btn>
            <v-progress-circular
              v-else
              indeterminate
              size="20"
              width="2"
              color="primary"
            ></v-progress-circular>
          </div>

          <v-alert
            v-if="showNoApiKeyMessage && useAiRecommendation"
            type="info"
            density="compact"
            variant="tonal"
            class="mt-2 mb-3"
            color="warning"
          >
            <div class="d-flex align-center">
              <span>OpenAI API Key is not configured. AI recommendations are disabled.</span>
            </div>
          </v-alert>
          <v-alert
            v-if="aiError && useAiRecommendation"
            type="error"
            density="compact"
            variant="tonal"
            class="mt-2 mb-3"
          >
            {{ aiError }}
          </v-alert>

          <div class="text-body-1">
            <div v-if="useAiRecommendation && aiRecommendation" v-html="formatRecommendation(aiRecommendation)"></div>
            <div v-else-if="useAiRecommendation && !aiRecommendation && !aiError && !showNoApiKeyMessage && loadingAiRec">Fetching AI insight...</div>
            <div v-else>{{ detailedRecommendation }}</div>
          </div>
        </v-card>

        <!-- Strategy Advice -->
        <v-card class="mb-6 pa-4">
          <div class="text-subtitle-1 font-weight-medium mb-3">Application Strategy</div>
          <v-list>
            <v-list-item v-if="isEarlyDecision">
              <template v-slot:prepend>
                <v-icon color="success" class="mr-2">mdi-check-circle</v-icon>
              </template>
              <v-list-item-title>Early Decision Benefit</v-list-item-title>
              <v-list-item-subtitle>
                Applying Early Decision can increase your chances by up to 20-30% at {{ college.name }}
              </v-list-item-subtitle>
            </v-list-item>
            <v-list-item v-if="isLegacy">
              <template v-slot:prepend>
                <v-icon color="success" class="mr-2">mdi-check-circle</v-icon>
              </template>
              <v-list-item-title>Legacy Status Benefit</v-list-item-title>
              <v-list-item-subtitle>
                Your legacy status improves your chances at {{ college.name }}
              </v-list-item-subtitle>
            </v-list-item>
            <v-list-item>
              <template v-slot:prepend>
                <v-icon :color="detailedCollegeChance.probabilityPercentage > 50 ? 'success' : 'warning'" class="mr-2">
                  {{ detailedCollegeChance.probabilityPercentage > 50 ? 'mdi-thumb-up' : 'mdi-alert' }}
                </v-icon>
              </template>
              <v-list-item-title>Major Match</v-list-item-title>
              <v-list-item-subtitle>
                {{ studentProfile.intendedMajor ? studentProfile.intendedMajor + ' is ' + (majorMatch === 'good' ? 'a good match' : majorMatch === 'neutral' ? 'an average match' : 'a challenging match') + ' for this school' : 'Add your intended major to see a match assessment' }}
              </v-list-item-subtitle>
            </v-list-item>
            <v-list-item v-if="detailedCollegeChance.probabilityPercentage < 30">
              <template v-slot:prepend>
                <v-icon color="error" class="mr-2">mdi-alert-circle</v-icon>
              </template>
              <v-list-item-title>Consider Adding Safer Options</v-list-item-title>
              <v-list-item-subtitle>
                This is a reach school. Make sure you have backup schools with higher admission chances.
              </v-list-item-subtitle>
            </v-list-item>
          </v-list>
        </v-card>

        <!-- Save Button -->
        <v-card class="mb-6 pa-4">
          <v-row>
            <v-col cols="12" sm="6">
              <v-btn
                color="primary"
                variant="elevated"
                block
                @click="$emit('saveToEarly', { college: college, action: 'add' })"
              >
                <v-icon left>mdi-star</v-icon>
                Save to Early Decision
              </v-btn>
            </v-col>
            <v-col cols="12" sm="6">
              <v-btn
                color="secondary"
                variant="elevated"
                block
                @click="$emit('saveToRegular', { college: college, action: 'add' })"
              >
                <v-icon left>mdi-content-save</v-icon>
                Save to Regular Decision
              </v-btn>
            </v-col>
          </v-row>
        </v-card>
      </v-card>
    </v-col>

    <v-col cols="12" md="4">
      <div class="sticky-top-side">
        <!-- Strength Metrics card is MOVED here -->
        <v-card class="mb-6 pa-4">
          <div class="text-h6 font-weight-bold mb-3">Key Profile Metrics</div> <!-- Optional: Renaming title for clarity in new position -->
          <v-row sm="6">
            <v-col cols="12" >
              <v-card variant="outlined" class="pa-3 mt-3">
                <div class="text-subtitle-1 font-weight-medium mb-2">Academic Strength</div>
                <v-progress-linear
                  :model-value="normalizeBlock(detailedCollegeChance.strengthBlock || 0)"
                  color="primary"
                  height="12"
                  class="mb-2"
                  rounded
                ></v-progress-linear>
                <div class="text-caption text-high-emphasis">
                  Your academic strength score: {{ formatNumber(detailedCollegeChance.strengthBlock || 0) }}
                </div>
              </v-card>
            </v-col>

          </v-row>
          <v-row sm="6">
            <v-col cols="12" >
              <v-card variant="outlined" class="pa-3 mb-3">
                <div class="text-subtitle-1 font-weight-medium mb-2">Mission Alignment</div>
                <v-progress-linear
                  :model-value="normalizeBlock(detailedCollegeChance.alignmentBlock || 0)"
                  color="secondary"
                  height="12"
                  class="mb-2"
                  rounded
                ></v-progress-linear>
                <div class="text-caption text-high-emphasis">
                  Your alignment score: {{ formatNumber(detailedCollegeChance.alignmentBlock || 0) }}
                </div>
              </v-card>
            </v-col>
          </v-row>
        </v-card>

        <!-- Additional Metrics and Details -->
        <v-card class="mb-6 pa-4">
          <div class="text-h6 font-weight-bold mb-3">Acceptance Rate Factors</div>
          <v-list density="compact">
            <v-list-item class="pa-0">
              <v-list-item-content class="py-2">
                <v-list-item-title class="text-subtitle-2 font-weight-medium">Base Acceptance Rate</v-list-item-title>
              </v-list-item-content>
              <template v-slot:append>
                <v-chip color="primary" size="small" variant="elevated">{{ college.acceptanceRate }}%</v-chip>
              </template>
            </v-list-item>

            <v-divider class="my-2"></v-divider>

            <v-list-item class="pa-0">
              <v-list-item-content class="py-2">
                <v-list-item-title class="text-subtitle-2 font-weight-medium">Major Adjusted Rate</v-list-item-title>
                <v-list-item-subtitle class="text-caption mt-1" v-if="studentProfile.intendedMajor">
                  {{ isHarderMajor ? 'Major is more competitive' : isEasierMajor ? 'Major improves chances' : 'Neutral major impact' }}
                </v-list-item-subtitle>
              </v-list-item-content>
              <template v-slot:append>
                <v-chip :color="isHarderMajor ? 'error' : isEasierMajor ? 'success' : 'primary'" size="small" variant="elevated">
                  {{ majorAdjustedRateForDisplay }}%
                </v-chip>
              </template>
            </v-list-item>

            <v-divider class="my-2"></v-divider>

            <v-list-item class="pa-0">
              <v-list-item-content class="py-2">
                <v-list-item-title class="text-subtitle-2 font-weight-medium">Final Adjusted Rate</v-list-item-title>
                <v-list-item-subtitle class="text-caption mt-1">
                  {{ isEarlyDecision ? 'ED provides a boost' : 'Regular Decision' }}
                  {{ isLegacy ? ' + Legacy status' : '' }}
                </v-list-item-subtitle>
              </v-list-item-content>
              <template v-slot:append>
                <v-chip :color="isEarlyDecision || isLegacy ? 'success' : 'primary'" size="small" variant="elevated">
                  {{ finalAdjustedRateForDisplay }}%
                </v-chip>
              </template>
            </v-list-item>
          </v-list>
        </v-card>



      </div>
    </v-col>
  </v-row>
</template>

<script setup>
import { computed } from 'vue';
// Updated imports to include all necessary functions
import {
  getAdmissionChanceColor,
  getAdmissionChanceDescription,
  getMajorMatchAssessment,
  adjustAcceptanceRateByMajor,
  adjustAcceptanceRateByStrategicFactors
} from '../../../utils/admitChanceCalculator';

const props = defineProps({
  college: { type: Object, required: true },
  isEarlyDecision: { type: Boolean, default: false },
  isLegacy: { type: Boolean, default: false },
  // Added missing props with safe defaults
  detailedCollegeChance: {
    type: Object,
    default: () => ({
      probabilityPercentage: 0,
      timesAverageApplicant: '0.0',
      zScores: {},
      strengthBlock: 0,
      alignmentBlock: 0,
      probability: 0
    })
  },
  studentProfile: {
    type: Object,
    default: () => ({
      intendedMajor: ''
    })
  },
  useAiRecommendation: { type: Boolean, default: false },
  loadingAiRec: { type: Boolean, default: false },
  showNoApiKeyMessage: { type: Boolean, default: false },
  aiError: { type: String, default: '' },
  aiRecommendation: { type: String, default: '' },
});

defineEmits([
  'update:isEarlyDecision',
  'update:isLegacy',
  'calculateDetailedChance',
  'saveToEarly',
  'saveToRegular',
  'toggleAiRecommendation',
]);

const detailedChanceColor = computed(() => {
  // Ensure props.detailedCollegeChance and its properties are accessed safely
  return getAdmissionChanceColor(props.detailedCollegeChance?.probability || 0);
});

const detailedChanceDescription = computed(() => {
  // Ensure props.detailedCollegeChance and its properties are accessed safely
  return getAdmissionChanceDescription(props.detailedCollegeChance?.probability || 0);
});

const detailedRecommendation = computed(() => {
  if (!props.detailedCollegeChance || !props.college) return 'Calculating recommendation...';

  const prob = props.detailedCollegeChance.probability || 0; // Safe access
  const collegeType = props.college.collegeType || 'this';
  const edStatus = props.isEarlyDecision ? 'Early Decision' : 'Regular Decision';

  if (prob < 0.1) {
    return `This is a very hard reach school in the ${collegeType} category. ${props.isEarlyDecision ? 'Even with Early Decision,' : ''} Apply only if it's your dream school and you have strong alternatives.`;
  } else if (prob < 0.3) {
    return `This is a reach school. Applying ${edStatus} to this ${collegeType} school requires highlighting your absolute best ${collegeType === 'STEM-heavy' ? 'STEM' : 'Liberal Arts'} strengths.`;
  } else if (prob < 0.5) {
    return `This is a possible target school. Your profile is competitive for this ${collegeType} school with ${edStatus}, but admission isn't certain. Strengthen your application.`;
  } else if (prob < 0.8) {
    return `You're a strong candidate. This ${collegeType} school with ${edStatus} is a good chance. Consider it a solid target.`;
  } else {
    return `This is a likely school for you. With your ${props.isEarlyDecision ? 'Early Decision application and ' : ''}strong profile, you have an excellent chance of admission.`;
  }
});

const majorMatch = computed(() => {
  if (!props.studentProfile || !props.studentProfile.intendedMajor || !props.college) {
    return 'neutral'; // Default if essential data is missing
  }
  const assessment = getMajorMatchAssessment(props.college, props.studentProfile.intendedMajor);
  if (assessment === 'Excellent' || assessment === 'Good') {
    return 'good';
  } else if (assessment === 'Fair') {
    return 'neutral';
  } else if (assessment === 'Poor') {
    return 'challenging';
  }
  return 'neutral'; // Fallback
});

// Added computed properties for display rates and major competitiveness
const majorAdjustedRateForDisplay = computed(() => {
  if (!props.college || !props.studentProfile) return props.college?.acceptanceRate?.toFixed(1) || '0.0';
  const rate = adjustAcceptanceRateByMajor(
    parseFloat(props.college.acceptanceRate),
    props.studentProfile.intendedMajor,
    props.college.collegeType
  );
  return rate.toFixed(1);
});

const finalAdjustedRateForDisplay = computed(() => {
  if (!props.college || !props.studentProfile || !props.detailedCollegeChance) return props.college?.acceptanceRate?.toFixed(1) || '0.0';
  // The finalAdjustedAcceptanceRateForP0 is already calculated in detailedCollegeChance if the structure is consistent
  // Otherwise, recalculate if necessary:
  let rate = adjustAcceptanceRateByMajor(
    parseFloat(props.college.acceptanceRate),
    props.studentProfile.intendedMajor,
    props.college.collegeType
  );
  rate = adjustAcceptanceRateByStrategicFactors(
    rate,
    props.isEarlyDecision,
    props.isLegacy,
    props.college.allowsEarlyDecision,
    props.college.considersLegacy
  );
  return rate.toFixed(1);
});

const isHarderMajor = computed(() => {
  if (!props.college || !props.studentProfile || !props.studentProfile.intendedMajor) return false;
  const baseRate = parseFloat(props.college.acceptanceRate);
  const majorAdjusted = adjustAcceptanceRateByMajor(
    baseRate,
    props.studentProfile.intendedMajor,
    props.college.collegeType
  );
  return majorAdjusted < baseRate * 0.9;
});

const isEasierMajor = computed(() => {
  if (!props.college || !props.studentProfile || !props.studentProfile.intendedMajor) return false;
  const baseRate = parseFloat(props.college.acceptanceRate);
  const majorAdjusted = adjustAcceptanceRateByMajor(
    baseRate,
    props.studentProfile.intendedMajor,
    props.college.collegeType
  );
  return majorAdjusted > baseRate * 1.1;
});


const formatZScore = (score) => {
  if (score === undefined || score === null) return '0.00';
  return score.toFixed(2);
};

const getZScoreColor = (score) => {
  if (score === undefined || score === null) score = 0;
  if (score > 1.5) return 'success';
  if (score > 0) return 'info';
  if (score > -1.5) return 'warning';
  return 'error';
};

const formatNumber = (num) => {
  if (num === undefined || num === null) return '0.00';
  return num.toFixed(2);
};

const normalizeBlock = (blockValue) => {
  if (blockValue === undefined || blockValue === null) return 0;
  return Math.min(Math.max((blockValue + 2) * 25, 0), 100);
};

const formatRecommendation = (text) => {
  if (!text) return '';
  return text
    .replace(/\n\n/g, '<br><br>')
    .replace(/\n/g, '<br>')
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    .replace(/\*(.*?)\*/g, '<em>$1</em>');
};

</script>

<style scoped>
.text-disabled {
  color: rgba(0, 0, 0, 0.38);
}

.sticky-top-side::-webkit-scrollbar {
  width: 6px;
}
.sticky-top-side::-webkit-scrollbar-thumb {
  background-color: #ccc;
  border-radius: 3px;
}
.sticky-top-side::-webkit-scrollbar-track {
  background-color: #f1f1f1;
}
</style>
