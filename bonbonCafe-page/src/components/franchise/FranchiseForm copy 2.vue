<template>
  <v-form ref="formRef" @submit.prevent="SubmitEvent">
    <v-container class="py-8">
      <v-card elevation="2" class="pa-6 rounded-xl" style="border: 1px solid #ccc;">
        <!-- 기본 정보 -->
        <v-row dense>
          <v-col cols="12" md="6">
            <v-row class="align-center mb-3">
              <v-col cols="4"><label>* 가맹점 이름</label></v-col>
              <v-col cols="8">
                <v-text-field v-model="form.name" :readonly="props.readonly" :rules="props.readonly ? [] : [v => !!v || '가맹점 이름은 필수입니다']" density="compact" variant="outlined" hide-details />
              </v-col>
            </v-row>

            <v-row class="align-center mb-3">
              <v-col cols="4"><label>* 가맹점 연락처</label></v-col>
              <v-col cols="8">
                <v-text-field v-model="form.franchiseTel" type="tel" :readonly="props.readonly" :rules="props.readonly ? [] : [v => !!v || '연락처는 필수입니다']" density="compact" variant="outlined" hide-details />
              </v-col>
            </v-row>

            <v-row class="align-center mb-1">
              <v-col cols="4"><label>* 주소</label></v-col>
              <v-col cols="8" class="pa-0">
                <template v-if="!props.readonly">
                  <KakaoAPI @address-selected="onAddressSelected" @update-detail-address="receiveDetailAddress" />
                </template>
                <template v-else>
                  <v-text-field v-model="form.roadAddress" label="주소" readonly density="compact" variant="outlined" hide-details class="mb-2" :rules="[v => !!v || '주소는 필수입니다']" />
                  <v-text-field v-model="form.detailAddress" label="상세주소" readonly density="compact" variant="outlined" hide-details />
                </template>
              </v-col>
            </v-row>
          </v-col>

          <v-col cols="12" md="6" class="d-flex flex-column align-center justify-center">
            <v-avatar size="120" class="mb-3">
              <v-img :src="franchiseImageUrl || defaultImage" cover />
            </v-avatar>
            <v-btn v-if="!props.readonly" color="grey" size="small" outlined @click="$refs.fileInput.click()">이미지 업로드</v-btn>
            <input ref="fileInput" type="file" accept="image/*" style="display:none" @change="onFranchiseImageChange($event.target.files)" />
          </v-col>
        </v-row>

        <!-- 하단 정보 -->
        <v-row dense>
          <v-col cols="12" md="4">
            <label>* 매장 평수</label>
            <v-text-field v-model="form.storeSize" type="number" :readonly="props.readonly" :rules="props.readonly ? [] : [v => !!v || '매장 평수는 필수입니다']" density="compact" variant="outlined" hide-details />
          </v-col>
          <v-col cols="12" md="4">
            <label>* 좌석수</label>
            <v-text-field v-model="form.seatingCapacity" type="number" :readonly="props.readonly" :rules="props.readonly ? [] : [v => !!v || '좌석 수는 필수입니다']" density="compact" variant="outlined" hide-details />
          </v-col>

          <v-col cols="12" md="4">
            <label>* 주차 여부</label>
            <v-radio-group v-model="form.parkingAvailability" row :disabled="props.readonly">
              <v-radio label="가능" :value="true" />
              <v-radio label="불가" :value="false" />
            </v-radio-group>
          </v-col>

          <v-col cols="12" md="4">
            <label>* 개업 일자</label>
            <template v-if="!props.readonly">
              <v-row dense>
                <v-col cols="4">
                  <v-select :items="years" v-model="form.openYear" label="년" density="compact" variant="outlined" :rules="[v => !!v || '년 선택']" />
                </v-col>
                <v-col cols="4">
                  <v-select :items="months" v-model="form.openMonth" label="월" density="compact" variant="outlined" :rules="[v => !!v || '월 선택']" />
                </v-col>
                <v-col cols="4">
                  <v-select :items="days" v-model="form.openDay" label="일" density="compact" variant="outlined" :rules="[v => !!v || '일 선택']" />
                </v-col>
              </v-row>
            </template>
            <template v-else>
              <div>{{ form.openDate }}</div>
            </template>
          </v-col>

          <v-col cols="12" md="4">
            <label>* 운영 상태</label>
            <v-radio-group v-model="form.status" row :disabled="props.readonly">
              <v-radio label="정상 운영" value="OPERATING" />
              <v-radio label="휴점" value="PAUSED" />
              <v-radio label="폐점" value="CLOSED" />
            </v-radio-group>
          </v-col>

          <v-col cols="12" md="4">
            <label>* 운영 시간</label>
            <template v-if="!props.readonly">
              <v-row dense>
                <v-col cols="5">
                  <v-text-field v-model="form.openTime" type="time" density="compact" variant="outlined" :rules="[v => !!v || '시작 시간 필수']" />
                </v-col>
                <v-col cols="2" class="d-flex align-center justify-center"><span>~</span></v-col>
                <v-col cols="5">
                  <v-text-field v-model="form.closeTime" type="time" density="compact" variant="outlined" :rules="[v => !!v || '종료 시간 필수']" />
                </v-col>
              </v-row>
            </template>
            <template v-else>
              <div>{{ form.openHours }}</div>
            </template>
          </v-col>
        </v-row>

        <v-row justify="center" v-if="props.submitVisible">
          <v-btn type="submit" color="primary" class="mt-6" size="large" rounded>제출</v-btn>
          <v-btn color="grey" class="mt-6 mx-2" size="large" rounded @click="emit('back')">뒤로가기</v-btn>
        </v-row>
        <!-- 읽기 전용일 때 하단 버튼 -->
        <v-row justify="center" v-if="props.readonly">
          <v-btn color="primary" class="mt-6 mx-2" size="large" rounded @click="emit('edit')">수정</v-btn>
          <v-btn color="error" class="mt-6 mx-2" size="large" rounded @click="emit('delete')">삭제</v-btn>
          <v-btn color="grey" class="mt-6 mx-2" size="large" rounded @click="emit('back')">뒤로가기</v-btn>
        </v-row>

      </v-card>
    </v-container>
  </v-form>
</template>

<script setup>
import { ref, reactive, watch } from 'vue'
import KakaoAPI from './KakaoAPI.vue'

const props = defineProps({
  initialFormData: Object,
  franchiseId: { type: [String, Number], required: false },  // 추가
  submitVisible: { type: Boolean, default: true },
  readonly: { type: Boolean, default: false },
})
const emit = defineEmits(['submit', 'edit', 'delete', 'back'])

const defaultImage = 'https://bonbon-file-bucket.s3.ap-northeast-2.amazonaws.com/profile-default.jpg'
const franchiseImageUrl = ref(null)
const fileInput = ref(null)
const formRef = ref(null)

const form = reactive({
  name: '',
  regionName: '',
  franchiseTel: '',
  roadAddress: '',
  detailAddress: '',
  openDate: '',
  openYear: '',
  openMonth: '',
  openDay: '',
  openTime: '',
  closeTime: '',
  openHours: '',
  franchiseImage: '',
  storeSize: 0,
  seatingCapacity: 0,
  parkingAvailability: false,
  status: 'OPERATING',
})

const years = ['2023', '2024', '2025']
const months = ['01','02','03','04','05','06','07','08','09','10','11','12']
const days = Array.from({ length: 31 }, (_, i) => String(i + 1).padStart(2, '0'))

watch(() => props.initialFormData, (newVal) => {
  if (newVal) {
    Object.assign(form, newVal)
    franchiseImageUrl.value = newVal.franchiseImageUrl || null
  }
}, { immediate: true })

function SubmitEvent() {
  if (props.readonly) return
  formRef.value.validate().then(success => {
    // if (success) {
    //   form.openDate = `${form.openYear}-${form.openMonth.padStart(2, '0')}-${form.openDay.padStart(2, '0')}`
    //   form.openHours = `${form.openTime}~${form.closeTime}`
    //   emit('submit', { ...form })
    // }

     const updatedata = {
        franchiseTel: form.franchiseTel,
        storeSize: form.storeSize,
        seatingCapacity: form.seatingCapacity,
        parkingAvailability: form.parkingAvailability,
        // openTime: form.openTime,
        // closeTime: form.closeTime,
        openHours: `${form.openTime}~${form.closeTime}`,
        status: form.status
      }
            emit('submit', updatedata)

  })
}

function onFranchiseImageChange(fileList) {
  const file = fileList instanceof FileList ? fileList[0] : fileList
  if (file) {
    franchiseImageUrl.value = URL.createObjectURL(file)
    form.franchiseImage = file.name
  } else {
    franchiseImageUrl.value = null
    form.franchiseImage = ''
  }
}

function onAddressSelected({ roadAddress, regionName }) {
  form.roadAddress = roadAddress
  form.regionName = regionName
}

function receiveDetailAddress(detailAddress) {
  form.detailAddress = detailAddress
}
</script>
