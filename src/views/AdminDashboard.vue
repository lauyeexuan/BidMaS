<template>
  <!-- Dashboard Content -->
  <div class="relative">
    <!-- Dashboard Cards -->
    <div class="mt-6 grid grid-cols-12 gap-4">
      <!-- Left Column -->
      <div class="col-span-6 flex flex-col gap-4">
        <!-- Milestone Card with Expandable Content -->
        <div 
          class="bg-white p-4 shadow rounded relative transition-all duration-200 overflow-hidden flex flex-col min-h-[160px]"
          :class="{'shadow-lg': showAllMilestones}"
        >
          <!-- Card Header -->
          <div>
            <!-- Major selector will be here -->
          </div>
          
          <div @click="toggleAllMilestones" class="cursor-pointer">
            <div class="flex justify-between items-center mb-2">
              <h2 class="text-sm font-medium text-gray-500 flex items-center">
                Current Milestone
                <svg 
                  xmlns="http://www.w3.org/2000/svg" 
                  class="h-4 w-4 ml-2 transition-transform duration-300" 
                  :class="{'rotate-180': showAllMilestones}"
                  fill="none" 
                  viewBox="0 0 24 24" 
                  stroke="currentColor"
                >
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                </svg>
              </h2>
              <!-- Major Selector Tabs -->
              <div class="flex space-x-2" @click.stop>
                <button
                  v-for="major in schoolMajors"
                  :key="major"
                  @click.stop="selectedMajor = major"
                  class="px-3 py-1 text-xs rounded-full transition-colors"
                  :class="selectedMajor === major ? 
                    'bg-blue-100 text-blue-800 font-medium' : 
                    'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                >
                  {{ major }}
                </button>
              </div>
            </div>
            
            <div v-if="loading" class="py-2">
              <div class="h-5 bg-gray-200 rounded animate-pulse w-3/4 mb-2"></div>
              <div class="h-4 bg-gray-200 rounded animate-pulse w-1/2"></div>
            </div>
            <div v-else-if="error" class="py-2">
              <p class="text-red-500">{{ error }}</p>
            </div>
            <div v-else-if="currentUpcomingMilestone" class="cursor-pointer">
              <!-- Milestone content with enhanced styling -->
              <div class="relative">
                <!-- Decorative element -->
                <div class="absolute left-0 top-0 bottom-0 w-1 bg-blue-500 rounded-full"></div>
                
                <div class="pl-4">
                  <h3 class="text-lg font-semibold text-blue-800 mb-1">{{ currentUpcomingMilestone.description }}</h3>
                  <div class="flex items-center text-gray-500">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                    </svg>
                    <span>{{ formatDate(currentUpcomingMilestone.deadline) }}</span>
                  </div>
                  
                  <!-- Days remaining indicator -->
                  <div class="mt-3 flex items-center">
                    <div class="w-full bg-gray-200 rounded-full h-2">
                      <div 
                        class="bg-blue-600 h-2 rounded-full" 
                        :style="`width: ${getDaysRemainingPercentage(currentUpcomingMilestone)}%`"
                      ></div>
                    </div>
                    <span class="ml-2 text-xs font-medium" :class="getDaysRemainingClass(currentUpcomingMilestone)">
                      {{ getDaysRemainingText(currentUpcomingMilestone) }}
                    </span>
                  </div>
                </div>
              </div>
            </div>
            <div v-else class="py-2">
              <p class="text-gray-500">No upcoming milestones found.</p>
            </div>
          </div>

          <!-- Expandable Milestone List -->
          <transition
            enter-active-class="transition-all duration-500 ease-out"
            enter-from-class="max-h-0 opacity-0 overflow-hidden"
            enter-to-class="max-h-[300px] opacity-100"
            leave-active-class="transition-all duration-300 ease-in"
            leave-from-class="max-h-[300px] opacity-100"
            leave-to-class="max-h-0 opacity-0 overflow-hidden"
          >
            <div v-if="showAllMilestones && filteredMilestones && filteredMilestones.length > 0" class="mt-4 pt-4 border-t border-gray-200 overflow-auto flex-grow">
              <div class="flex justify-between items-center mb-3">
                <h3 class="font-medium text-gray-800">All Milestones</h3>
              </div>
              
              <div class="milestone-list-container">
                <transition-group 
                  name="milestone-list" 
                  tag="div"
                  class="space-y-3"
                >
                  <div 
                    v-for="(milestone, index) in otherMilestones" 
                    :key="index"
                    class="p-3 rounded-md transition-all duration-200 relative overflow-hidden"
                    :class="isMilestonePast(milestone) ? 'bg-gray-50 hover:bg-gray-100' : 'bg-blue-50 hover:bg-blue-100'"
                  >
                    <!-- Decorative element -->
                    <div 
                      class="absolute left-0 top-0 bottom-0 w-1 rounded-full"
                      :class="isMilestonePast(milestone) ? 'bg-gray-400' : 'bg-blue-500'"
                    ></div>
                    
                    <div class="flex justify-between items-start pl-3">
                      <div>
                        <p class="font-medium" :class="isMilestonePast(milestone) ? 'text-gray-600' : 'text-blue-700'">
                          {{ milestone.description }}
                        </p>
                        <p class="text-sm flex items-center mt-1" :class="isMilestonePast(milestone) ? 'text-gray-500' : 'text-blue-600'">
                          <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                          </svg>
                          {{ formatDate(milestone.deadline) }}
                        </p>
                      </div>
                      <div 
                        class="text-xs px-2 py-1 rounded-full"
                        :class="isMilestonePast(milestone) ? 'bg-gray-200 text-gray-700' : 'bg-blue-200 text-blue-800'"
                      >
                        {{ isMilestonePast(milestone) ? 'Past' : 'Upcoming' }}
                      </div>
                    </div>
                  </div>
                </transition-group>
              </div>
            </div>
          </transition>
        </div>

        <!-- Submission Info Card (Now below milestone card) -->
        <div class="bg-white p-4 shadow rounded relative">
          <div class="flex justify-between items-center mb-3">
            <h2 class="text-sm font-medium text-gray-500">Milestone Submissions</h2>
          </div>
          
          <div v-if="submissionLoading" class="py-4">
            <div class="h-6 bg-gray-200 rounded animate-pulse w-3/4 mb-3"></div>
            <div class="h-4 bg-gray-200 rounded animate-pulse w-1/2 mb-2"></div>
            <div class="h-4 bg-gray-200 rounded animate-pulse w-2/3"></div>
          </div>
          
          <div v-else-if="submissionError" class="py-4">
            <p class="text-red-500">{{ submissionError }}</p>
          </div>
          
          <div v-else-if="currentMilestoneSubmissionStats" class="py-2">
            <div class="relative">
              <div class="absolute left-0 top-0 bottom-0 w-1 bg-purple-500 rounded-full"></div>
              
              <div class="pl-4">
                <h3 class="text-lg font-semibold text-gray-800 mb-3">
                  {{ currentMilestoneSubmissionStats.milestoneName }}
                </h3>
                
                <!-- Circular Progress Display -->
                <div class="flex flex-col items-center justify-center py-6">
                  <div class="relative w-32 h-32">
                    <!-- Background Circle -->
                    <svg class="w-full h-full" viewBox="0 0 100 100">
                      <circle
                        cx="50"
                        cy="50"
                        r="45"
                        fill="none"
                        stroke="#E2E8F0"
                        stroke-width="8"
                      />
                      <!-- Progress Circle -->
                      <circle
                        cx="50"
                        cy="50"
                        r="45"
                        fill="none"
                        stroke="#7C3AED"
                        stroke-width="8"
                        stroke-linecap="round"
                        :stroke-dasharray="`${currentMilestoneSubmissionStats.submissionRate * 2.83} 283`"
                        transform="rotate(-90 50 50)"
                      />
                    </svg>
                    <!-- Fraction Display -->
                    <div class="absolute inset-0 flex items-center justify-center">
                      <span class="text-2xl font-bold text-gray-800">
                        {{ currentMilestoneSubmissionStats.projectsWithSubmissions }}/{{ currentMilestoneSubmissionStats.totalAssigned }}
                      </span>
                    </div>
                  </div>
                  <!-- Submission Status Text -->
                  <p class="mt-4 text-center text-gray-600">
                    {{ currentMilestoneSubmissionStats.projectsWithSubmissions }} out of {{ currentMilestoneSubmissionStats.totalAssigned }} 
                    {{ currentMilestoneSubmissionStats.totalAssigned === 1 ? 'student' : 'students' }}
                    {{ currentMilestoneSubmissionStats.projectsWithSubmissions < 2 ? 'has' : 'have' }} submitted their work.
                  </p>
                </div>
              </div>
            </div>
          </div>
          
          <div v-else class="py-4 text-center">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto text-gray-400 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" />
            </svg>
            <p class="text-gray-500">No submission data available.</p>
            <p class="text-sm text-gray-400 mt-1">
              {{ currentUpcomingMilestone ? 'No assigned projects for this milestone yet.' : 'No current milestone found.' }}
            </p>
          </div>
        </div>
      </div>

      <!-- Right Column - Project Assignment Card -->
      <div class="col-span-6">
        <div class="bg-white p-4 shadow rounded self-start min-h-[160px] relative">
          <div class="flex justify-between items-start">
            <h2 class="text-sm font-medium text-gray-500 mb-2">Projects Overview</h2>
            
            <!-- Major Selector Tabs for Projects -->
            <div class="flex space-x-2">
              <button
                v-for="major in schoolMajors"
                :key="major"
                @click="selectedProjectMajor = major"
                class="px-3 py-1 text-xs rounded-full transition-colors"
                :class="selectedProjectMajor === major ? 
                  'bg-green-100 text-green-800 font-medium' : 
                  'bg-gray-100 text-gray-600 hover:bg-gray-200'"
              >
                {{ major }}
              </button>
            </div>
          </div>
          
          <div v-if="projectLoading" class="py-4">
            <div class="h-6 bg-gray-200 rounded animate-pulse w-3/4 mb-3"></div>
            <div class="h-4 bg-gray-200 rounded animate-pulse w-1/2 mb-2"></div>
            <div class="h-4 bg-gray-200 rounded animate-pulse w-2/3"></div>
          </div>
          
          <div v-else-if="projectError" class="py-4">
            <p class="text-red-500">{{ projectError }}</p>
          </div>
          
          <div v-else-if="currentMajorProjectStats && currentMajorProjectStats.total > 0" class="py-2">
            <div class="relative">
              <div class="absolute left-0 top-0 bottom-0 w-1 bg-green-500 rounded-full"></div>
              
              <div class="pl-4">
                <h3 class="text-lg font-semibold text-gray-800 mb-3">{{ selectedProjectMajor }} Projects</h3>
                
                <div class="grid grid-cols-3 gap-4 mb-4">
                  <!-- Total Projects -->
                  <div class="bg-blue-50 p-3 rounded-lg text-center">
                    <p class="text-2xl font-bold text-blue-700">{{ currentMajorProjectStats.total }}</p>
                    <p class="text-sm text-blue-600">Total Projects</p>
                  </div>
                  
                  <!-- Assigned Projects -->
                  <div class="bg-green-50 p-3 rounded-lg text-center">
                    <p class="text-2xl font-bold text-green-700">{{ currentMajorProjectStats.assigned }}</p>
                    <p class="text-sm text-green-600">Assigned</p>
                  </div>
                  
                  <!-- Unassigned Projects -->
                  <div class="bg-amber-50 p-3 rounded-lg text-center">
                    <p class="text-2xl font-bold text-amber-700">{{ currentMajorProjectStats.unassigned }}</p>
                    <p class="text-sm text-amber-600">Unassigned</p>
                  </div>
                </div>
                
                <!-- Assignment Rate Progress Bar -->
                <div class="mt-2">
                  <div class="flex justify-between items-center mb-1">
                    <span class="text-sm font-medium text-gray-700">Assignment Rate</span>
                    <span class="text-sm font-medium text-gray-700">{{ currentMajorProjectStats.assignmentRate }}%</span>
                  </div>
                  <div class="w-full bg-gray-200 rounded-full h-2.5">
                    <div 
                      class="bg-blue-600 h-2.5 rounded-full" 
                      :style="`width: ${currentMajorProjectStats.assignmentRate}%`"
                    ></div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          
          <div v-else class="py-4 text-center">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto text-gray-400 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
            </svg>
            <p class="text-gray-500">No projects data available for {{ selectedProjectMajor }}.</p>
            <p class="text-sm text-gray-400 mt-1">Projects may not have been created yet.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed, watch } from 'vue'
import { useUserStore } from '@/stores/userStore'
import { getMilestones } from '@/utils/milestones'
import { getLatestAcademicYear } from '@/utils/latestAcademicYear'
import { db } from '@/firebase'
import { collection, getDocs, query, where, doc, getDoc, limit } from 'firebase/firestore'

export default {
  setup() {
    const userStore = useUserStore()
    const allMilestones = ref([])
    const loading = ref(true)
    const error = ref(null)
    const showAllMilestones = ref(false)
    const schoolMajors = ref([])
    const selectedMajor = ref(null)
    
    // Submission data
    const submissionLoading = ref(true)
    const submissionError = ref(null)
    const selectedSubmissionMajor = ref(null)
    const currentMilestoneSubmissionStats = ref(null)
    const submissionStatsCache = ref({})
    const initialLoadDone = ref(false)

    // Project data
    const projectLoading = ref(true)
    const projectError = ref(null)
    const selectedProjectMajor = ref(null)
    const currentMajorProjectStats = ref(null)

    // In the setup function, add a projectStatsCache ref
    const projectStatsCache = ref({})

    // Computed property to filter milestones based on selected major
    const filteredMilestones = computed(() => {
      if (!allMilestones.value) return []
      if (!selectedMajor.value) return []
      
      return allMilestones.value.filter(milestone => 
        milestone.major === selectedMajor.value
      )
    })

    // Computed property for the upcoming milestone
    const currentUpcomingMilestone = computed(() => {
      console.log('Computing currentUpcomingMilestone')
      console.log('filteredMilestones.value:', filteredMilestones.value)
      
      if (!filteredMilestones.value || filteredMilestones.value.length === 0) {
        console.log('No filtered milestones available')
        return null
      }
      
      const now = new Date()
      console.log('Current time:', now)
      
      const sortedMilestones = [...filteredMilestones.value].sort((a, b) => {
        const dateA = a.deadline instanceof Date ? a.deadline : a.deadline.toDate()
        const dateB = b.deadline instanceof Date ? b.deadline : b.deadline.toDate()
        return dateA - dateB
      })
      console.log('Sorted milestones:', sortedMilestones)
      
      // Find the first upcoming milestone for the selected major
      const upcoming = sortedMilestones.find(milestone => {
        const deadlineDate = milestone.deadline instanceof Date ? 
          milestone.deadline : 
          milestone.deadline.toDate()
        console.log('Checking milestone:', milestone.description, 'deadline:', deadlineDate)
        const isUpcoming = deadlineDate > now
        console.log('Is upcoming:', isUpcoming)
        return isUpcoming
      })
      
      console.log('Found upcoming milestone:', upcoming)
      
      // If no upcoming milestone, use the most recent one
      const result = upcoming || sortedMilestones[sortedMilestones.length - 1]
      console.log('Final selected milestone:', result)
      return result
    })

    // Computed property to filter out the current milestone from the list
    const otherMilestones = computed(() => {
      if (!filteredMilestones.value || !currentUpcomingMilestone.value) return []
      
      return filteredMilestones.value.filter(milestone => 
        milestone.description !== currentUpcomingMilestone.value.description ||
        milestone.major !== currentUpcomingMilestone.value.major
      )
    })

    // Helper functions
    const toggleAllMilestones = () => {
      showAllMilestones.value = !showAllMilestones.value
    }

    const isMilestonePast = (milestone) => {
      if (!milestone || !milestone.deadline) return false
      
      const deadlineDate = milestone.deadline instanceof Date ? 
        milestone.deadline : 
        milestone.deadline.toDate()
      
      return new Date() > deadlineDate
    }

    const getDaysRemaining = (milestone) => {
      if (!milestone || !milestone.deadline) return 0
      
      const deadlineDate = milestone.deadline instanceof Date ? 
        milestone.deadline : 
        milestone.deadline.toDate()
      
      const now = new Date()
      
      // If deadline has passed, return 0
      if (now > deadlineDate) return 0
      
      // Calculate days remaining
      const diffTime = Math.abs(deadlineDate - now)
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
      
      return diffDays
    }

    const getDaysRemainingPercentage = (milestone) => {
      const daysRemaining = getDaysRemaining(milestone)
      return daysRemaining <= 30 ? (daysRemaining / 30) * 100 : 100
    }

    const getDaysRemainingText = (milestone) => {
      const days = getDaysRemaining(milestone)
      if (days === 0) return 'Due today'
      if (days === 1) return '1 day left'
      return `${days} days left`
    }

    const getDaysRemainingClass = (milestone) => {
      const days = getDaysRemaining(milestone)
      if (days < 3) return 'text-red-600'
      if (days < 7) return 'text-orange-500'
      return 'text-green-600'
    }

    const formatDate = (date) => {
      if (!date) return ''
      try {
        const dateObj = date instanceof Date ? date : date.toDate()
        return dateObj.toLocaleString('en-US', {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        })
      } catch (err) {
        return 'Invalid date'
      }
    }

    // Helper function to get major document ID
    const getMajorDocId = async (schoolId, yearId, majorId) => {
      try {
        console.log(`Getting major doc ID for ${majorId} in year ${yearId}`)
        const majorRef = collection(db, 'schools', schoolId, 'projects', yearId, majorId)
        const majorDocsQuery = query(majorRef, limit(1))
        const majorDocsSnapshot = await getDocs(majorDocsQuery)
        
        if (majorDocsSnapshot.empty) {
          console.log(`No documents found for major ${majorId}`)
          return null
        }
        
        const docId = majorDocsSnapshot.docs[0].id
        console.log(`Found major doc ID for ${majorId}:`, docId)
        return docId
      } catch (err) {
        console.error(`Error getting major doc ID for ${majorId}:`, err)
        return null
      }
    }

    // Add the fetchMilestonesData function
    const fetchMilestonesData = async () => {
      loading.value = true
      error.value = null
      
      try {
        if (!userStore.currentUser?.school) {
          throw new Error('School information not found')
        }

        const school = userStore.currentUser.school
        console.log('Fetching milestones for school:', school)
        
        const academicYearData = await getLatestAcademicYear(school)
        console.log('Academic year data:', academicYearData)
        
        if (!academicYearData?.yearId) {
          throw new Error('Failed to determine academic year')
        }

        const yearId = academicYearData.yearId
        console.log('Using year ID:', yearId)

        // Get all majors first if not already loaded
        if (schoolMajors.value.length === 0) {
          await fetchSchoolMajors()
        }
        console.log('Available majors:', schoolMajors.value)
        
        // Create an array of promises to get all milestones in parallel
        const milestonesPromises = schoolMajors.value.map(async majorId => {
          try {
            console.log(`Fetching milestones for major: ${majorId}`)
            // First get the major document ID
            const majorCollectionRef = collection(db, 'schools', school, 'projects', yearId, majorId)
            const majorDocsQuery = query(majorCollectionRef, limit(1))
            const majorDocsSnapshot = await getDocs(majorDocsQuery)
            
            if (majorDocsSnapshot.empty) {
              console.log(`No documents found for major ${majorId}`)
              return []
            }
            
            const majorDoc = majorDocsSnapshot.docs[0]
            console.log(`Found major document for ${majorId}:`, majorDoc.id)
            const majorData = majorDoc.data()
            console.log(`Major data for ${majorId}:`, majorData)
            
            // Get milestones array from the document data
            const milestones = majorData.milestones || []
            console.log(`Raw milestones for ${majorId}:`, milestones)
            
            // Map the milestones and add the major ID
            const processedMilestones = milestones.map(milestone => ({
              ...milestone,
              major: majorId
            }))
            console.log(`Processed milestones for ${majorId}:`, processedMilestones)
            return processedMilestones
          } catch (err) {
            console.error(`Error fetching milestones for major ${majorId}:`, err)
            return []
          }
        })

        // Wait for all milestone queries to complete
        const milestonesArrays = await Promise.all(milestonesPromises)
        console.log('All milestone arrays:', milestonesArrays)
        
        // Combine all milestones into a single array
        const combinedMilestones = milestonesArrays.flat()
        console.log('Combined milestones:', combinedMilestones)
        
        // Update the allMilestones ref
        allMilestones.value = combinedMilestones
        console.log('Updated allMilestones.value:', allMilestones.value)

      } catch (err) {
        console.error('Error in fetchMilestonesData:', err)
        error.value = `Failed to load milestone data: ${err.message}`
      } finally {
        loading.value = false
      }
    }

    // Fetch all majors for the school
    const fetchSchoolMajors = async () => {
      try {
        if (!userStore.currentUser?.school) {
          throw new Error('School information not found')
        }

        const school = userStore.currentUser.school
        const academicYearData = await getLatestAcademicYear(school)
        
        if (!academicYearData?.yearId) {
          throw new Error('Failed to determine academic year')
        }

        // Get the academic year document which contains the majors array
        const yearRef = doc(db, 'schools', school, 'projects', academicYearData.yearId)
        const yearDoc = await getDoc(yearRef)
        
        if (!yearDoc.exists()) {
          throw new Error('Academic year document not found')
        }

        const yearData = yearDoc.data()
        const majors = yearData.majors || []
        console.log('Found majors in academic year document:', majors)
        
        schoolMajors.value = majors

        // Set default selected major
        if (majors.length > 0 && !selectedMajor.value) {
          selectedMajor.value = majors[0]
          selectedSubmissionMajor.value = majors[0]
        }

        return majors
      } catch (err) {
        console.error('Error fetching school majors:', err)
        error.value = 'Failed to load school majors'
        throw err
      }
    }

    // Function to fetch submission statistics for the current milestone
    const fetchSubmissionStats = async (majorId, isBackgroundLoad = false) => {
      if (!majorId) return
      
      console.log('Starting fetchSubmissionStats for major:', majorId, 'background:', isBackgroundLoad)
      
      if (!isBackgroundLoad) {
        selectedSubmissionMajor.value = majorId
      }
      
      // Check cache
      if (submissionStatsCache.value[majorId]) {
        console.log('Using cached submission stats for major:', majorId)
        if (!isBackgroundLoad) {
          currentMilestoneSubmissionStats.value = submissionStatsCache.value[majorId]
        }
        return
      }
      
      if (!isBackgroundLoad) {
        submissionLoading.value = true
        submissionError.value = null
        currentMilestoneSubmissionStats.value = null
      }
      
      try {
        if (!userStore.currentUser?.school) {
          throw new Error('School information not found')
        }
        
        const school = userStore.currentUser.school
        const academicYearData = await getLatestAcademicYear(school)
        
        if (!academicYearData?.yearId) {
          throw new Error('Failed to determine academic year')
        }
        
        const yearId = academicYearData.yearId
        const majorDocId = await getMajorDocId(school, yearId, majorId)
        
        if (!majorDocId) {
          throw new Error('Major document not found')
        }
        
        // Get all projects for this major
        const projectsRef = collection(
          db,
          'schools', school,
          'projects', yearId,
          majorId, majorDocId,
          'projectsPerYear'
        )
        
        const projectsSnapshot = await getDocs(projectsRef)
        
        // Get current milestone index
        const majorMilestones = await getMilestones(school, yearId, majorId, majorDocId)
        const now = new Date()
        const sortedMilestones = [...majorMilestones].sort((a, b) => {
          const dateA = a.deadline instanceof Date ? a.deadline : a.deadline.toDate()
          const dateB = b.deadline instanceof Date ? b.deadline : b.deadline.toDate()
          return dateA - dateB
        })
        
        let currentMilestone = sortedMilestones.find(milestone => {
          const deadlineDate = milestone.deadline instanceof Date ? 
            milestone.deadline : 
            milestone.deadline.toDate()
          return deadlineDate > now
        })
        
        if (!currentMilestone && sortedMilestones.length > 0) {
          currentMilestone = sortedMilestones[sortedMilestones.length - 1]
        }
        
        if (!currentMilestone) {
          throw new Error('No current milestone found')
        }
        
        const milestoneIndex = majorMilestones.findIndex(m => 
          m.description === currentMilestone.description
        )
        
        // Count assigned projects and prepare submission queries
        let totalAssigned = 0
        const submissionQueries = []
        
        projectsSnapshot.docs.forEach(projectDoc => {
          const projectData = projectDoc.data()
          if (projectData.assignedTo) {
            totalAssigned++
            const submissionsRef = collection(projectDoc.ref, 'submissions')
            const submissionsQuery = query(
              submissionsRef,
              where('milestoneIndex', '==', milestoneIndex)
            )
            submissionQueries.push({
              projectId: projectDoc.id,
              query: submissionsQuery
            })
          }
        })
        
        // Execute all submission queries in parallel
        const submissionResults = await Promise.all(
          submissionQueries.map(item => 
            getDocs(item.query).then(snapshot => ({
              projectId: item.projectId,
              hasSubmission: !snapshot.empty
            }))
          )
        )
        
        const projectsWithSubmissions = submissionResults.filter(result => result.hasSubmission).length
        const submissionRate = totalAssigned > 0 ? 
          Math.round((projectsWithSubmissions / totalAssigned) * 100) : 0
        
        const statsObject = {
          milestoneName: currentMilestone.description,
          milestoneIndex: milestoneIndex,
          totalAssigned: totalAssigned,
          projectsWithSubmissions: projectsWithSubmissions,
          projectsWithoutSubmissions: totalAssigned - projectsWithSubmissions,
          submissionRate: submissionRate,
          timestamp: Date.now()
        }
        
        submissionStatsCache.value[majorId] = statsObject
        
        if (!isBackgroundLoad) {
          currentMilestoneSubmissionStats.value = statsObject
        }
        
      } catch (err) {
        if (!isBackgroundLoad) {
          submissionError.value = `Failed to load submission data: ${err.message}`
        }
        console.error(`Error loading submission stats for major ${majorId}:`, err)
      } finally {
        if (!isBackgroundLoad) {
          submissionLoading.value = false
        }
      }
    }

    // Function to pre-load submission statistics for all majors
    const preloadAllSubmissionStats = async () => {
      if (!schoolMajors.value || schoolMajors.value.length === 0) return
      
      console.log('Pre-loading submission statistics for all majors')
      
      const fetchPromises = schoolMajors.value
        .filter(majorId => majorId !== selectedMajor.value)
        .map(majorId => {
          if (!submissionStatsCache.value[majorId]) {
            console.log('Preloading stats for major:', majorId)
            return fetchSubmissionStats(majorId, true)
          }
          return Promise.resolve()
        })
      
      Promise.all(fetchPromises)
        .then(() => console.log('Finished pre-loading submission statistics'))
        .catch(err => console.error('Error during background preloading:', err))
    }

    // Add watcher for selectedMajor
    watch(selectedMajor, async (newMajor, oldMajor) => {
      if (!newMajor) return
      
      if (!initialLoadDone.value) {
        return
      }

      console.log('Selected major changed to:', newMajor)
      await fetchSubmissionStats(newMajor)
    })

    // Function to fetch project statistics for a specific major
    const fetchProjectStats = async (majorId, isBackgroundLoad = false) => {
      if (!majorId) return
      
      console.log('Starting fetchProjectStats for major:', majorId, 'background:', isBackgroundLoad)
      
      // Check cache first
      if (projectStatsCache.value[majorId]) {
        console.log('Using cached project stats for major:', majorId)
        currentMajorProjectStats.value = projectStatsCache.value[majorId]
        return
      }
      
      if (!isBackgroundLoad) {
        projectLoading.value = true
        projectError.value = null
      }
      
      try {
        if (!userStore.currentUser?.school) {
          throw new Error('School information not found')
        }
        
        const school = userStore.currentUser.school
        const academicYearData = await getLatestAcademicYear(school)
        
        if (!academicYearData?.yearId) {
          throw new Error('Failed to determine academic year')
        }
        
        const yearId = academicYearData.yearId
        const majorDocId = await getMajorDocId(school, yearId, majorId)
        
        if (!majorDocId) {
          throw new Error('Major document not found')
        }
        
        // Get all projects for this major
        const projectsRef = collection(
          db,
          'schools', school,
          'projects', yearId,
          majorId, majorDocId,
          'projectsPerYear'
        )
        
        const projectsSnapshot = await getDocs(projectsRef)
        
        // Count total, assigned and unassigned projects
        let totalProjects = projectsSnapshot.size
        let assignedProjects = 0
        
        projectsSnapshot.forEach(doc => {
          const projectData = doc.data()
          if (projectData.assignedTo) {
            assignedProjects++
          }
        })
        
        const unassignedProjects = totalProjects - assignedProjects
        const assignmentRate = totalProjects > 0 ? Math.round((assignedProjects / totalProjects) * 100) : 0
        
        // Create stats object
        const statsObject = {
          total: totalProjects,
          assigned: assignedProjects,
          unassigned: unassignedProjects,
          assignmentRate: assignmentRate,
          timestamp: Date.now()
        }
        
        // Store in cache
        projectStatsCache.value[majorId] = statsObject
        
        // Update current stats
        if (majorId === selectedProjectMajor.value || !selectedProjectMajor.value) {
          currentMajorProjectStats.value = statsObject
        }
        
        console.log(`Project stats for ${majorId}:`, statsObject)
        
      } catch (err) {
        if (!isBackgroundLoad) {
          projectError.value = `Failed to load project data: ${err.message}`
        }
        console.error(`Error loading project stats for major ${majorId}:`, err)
      } finally {
        if (!isBackgroundLoad) {
          projectLoading.value = false
        }
      }
    }

    // Function to preload project stats for all majors
    const preloadAllProjectStats = async () => {
      if (!schoolMajors.value || schoolMajors.value.length === 0) return
      
      console.log('Pre-loading project statistics for all majors')
      
      const fetchPromises = schoolMajors.value.map(majorId => {
        return fetchProjectStats(majorId, true)
      })
      
      Promise.all(fetchPromises)
        .then(() => console.log('Finished pre-loading project statistics'))
        .catch(err => console.error('Error during background preloading of projects:', err))
    }

    // Update the watcher for selectedProjectMajor to use the cache
    watch(selectedProjectMajor, async (newMajor, oldMajor) => {
      if (!newMajor) return
      
      console.log('Selected project major changed to:', newMajor)
      
      // Check if we have cached data
      if (projectStatsCache.value[newMajor]) {
        console.log('Using cached project stats for major:', newMajor)
        currentMajorProjectStats.value = projectStatsCache.value[newMajor]
      } else {
        // Fetch if not in cache
        await fetchProjectStats(newMajor)
      }
    })

    onMounted(async () => {
      console.log('AdminDashboard mounted')
      const startTime = Date.now()
      try {
        if (!userStore.initialized) {
          await userStore.initializeAuth()
        }

        // First fetch all majors for the school
        await fetchSchoolMajors()

        // Set default selected project major if not already set
        if (schoolMajors.value.length > 0 && !selectedProjectMajor.value) {
          selectedProjectMajor.value = schoolMajors.value[0]
        }

        // Then fetch milestone data, submission stats, and project stats in parallel
        await Promise.all([
          fetchMilestonesData(),
          selectedMajor.value ? fetchSubmissionStats(selectedMajor.value) : Promise.resolve(),
          selectedProjectMajor.value ? fetchProjectStats(selectedProjectMajor.value) : Promise.resolve()
        ])

        initialLoadDone.value = true

        // Start preloading other majors' stats
        if (selectedMajor.value) {
          preloadAllSubmissionStats()
        }
        
        // Preload project stats for all majors
        preloadAllProjectStats()

        const totalLoadTime = Date.now() - startTime
        console.log(`Total dashboard load time: ${totalLoadTime}ms`)
      } catch (err) {
        console.error('Failed to initialize dashboard:', err)
        error.value = 'Failed to initialize dashboard data'
      } finally {
        loading.value = false
        projectLoading.value = false
      }
    })

    return {
      allMilestones,
      loading,
      error,
      showAllMilestones,
      schoolMajors,
      selectedMajor,
      submissionLoading,
      submissionError,
      currentMilestoneSubmissionStats,
      currentUpcomingMilestone,
      filteredMilestones,
      otherMilestones,
      toggleAllMilestones,
      isMilestonePast,
      getDaysRemaining,
      getDaysRemainingPercentage,
      getDaysRemainingText,
      getDaysRemainingClass,
      formatDate,
      selectedSubmissionMajor,
      preloadAllSubmissionStats,
      fetchSubmissionStats,
      projectLoading,
      projectError,
      selectedProjectMajor,
      currentMajorProjectStats,
      fetchProjectStats
    }
  }
}
</script>

<style scoped>
/* Dashboard styles imported from assets/styles/dashboard.css */
</style>

