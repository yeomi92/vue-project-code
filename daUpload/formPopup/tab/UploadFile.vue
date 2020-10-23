<template>
  <div>
    <input-row
      rowName="이미지 업로드"
      :tooltip="{
        text: `이미지 업로드 툴팁`
      }"
    >
      <div class="row-description-text">
        [이미지 파일 찾기] 버튼을 눌러 소재 이미지를 등록해주세요.
      </div>
      <div class="row-sub-description-text">
        이미지 검수 완료 후 성공한 결과만 리스트에 나타납니다.<br>
        검수에 실패한 파일이 존재할 경우, 최근 검수 실패를 클릭하시면, 자세한 내용을 확인하실 수 있습니다.<br>
        (최근 작업의 실패 내역만 나타납니다.)
      </div>
      <div class="common__list-layout">
        <div class="common__list-layout__header">
          <click-button
            text="이미지 파일 찾기"
            bgColor="blue"
            :fileButton="true"
            :fileButtonOption="{
              multiple: true,
              accept: 'image/*'
            }"
            @selectFile="selectImageFile"
          />
          <click-button
            text="제거"
            bgColor="point-red"
            :disabled="checkedList.length === 0"
          />
          <div class="common__list-layout__header--right-push">
            최근 검수 실패 : <span class="underline-text">{{failImageList.length}}개</span>
          </div>
        </div>
        <div class="common__list-layout__section">
          <list-table
            indexKey="id"
            :data="imageList"
            :showRowNum="3"
            useRowButton="checkbox"
          >
            <template v-slot="{ row }">
              <list-table-column :label="`검수 성공 이미지 파일${imageList.length === 0 ? '' : `(총 ${imageList.length}개)`}`" :width="400" align="center">
                {{row.name}}
              </list-table-column>
              <list-table-column label="이미지 용량" width="auto" align="center">
                {{row.size/1000}}KB
              </list-table-column>
            </template>
            <template v-slot:noData>
              <div>
                이미지 파일을 선택해주세요.
              </div>
            </template>
          </list-table>
        </div>
      </div>
    </input-row>
    <input-row
      rowName="파일 업로드"
      :tooltip="{
        text: `파일 업로드 툴팁`
      }"
    >
      <grid-flex align="center">
        <input-radios
          v-model="uploadType"
          :options="[
            { text: '엑셀 업로드하기', value: 'excel' },
            { text: '직접 입력하기', value: 'direct' }
          ]"
          :block="false"
          gap="40"
          @change="changeUploadType"
        />
        <click-button
          v-if="uploadType === 'direct'"
          style="margin-left: 15px;"
          bgColor="blue"
          :text="'입력'"
          height="30"
          @click="showPopup = true"
        />
      </grid-flex>
      <div class="row-description-text" v-if="uploadType === 'excel'">
        템플릿 파일을 다운로드 하신 후, [파일 찾기] 버튼을 눌러 업로드 파일을 등록해주세요.
      </div>
      <div class="inline-row" v-if="uploadType === 'excel'">
        <input-field-v2
          class="work-unit-input-field"
          v-model="uploadFile.name"
          :placeholder="!!uploadFile ? '업로드할 파일을 선택해주세요.' : '작업을 먼저 선택해주세요'"
          disabled
        />
        <click-button
          text="파일 찾기"
          bgColor="blue"
          :fileButton="true"
          @selectFile="selectFile"
        />
        <click-button
          text="템플릿 파일 다운"
        />
      </div>
    </input-row>
  </div>
</template>

<script>
const DISPLAY_TYPE_LIST = ['jpg', 'jpeg', 'png']
const BIZ_TYPE_LIST = ['png-24bit']
const LIMIT_COUNT = 250
export default {
  name: 'DaUploadDaUploadSubpopupUploadFile',
  data () {
    return {
      imageList: [],
      failImageList: [],
      uploadType: 'excel',
      uploadFile: {
        name: '',
        size: 0,
        file: null
      },
      showPopup: false
    }
  },
  computed: {
    checkedList () {
      return this.imageList.filter(i => i.checked)
    }
  },
  methods: {
    changeUploadType () {
      console.log('업로드 타입변경')
    },
    selectImageFile ({ files }) {
      const fileList = []
      const campaignType = 'display'
      let existsNotCsv = false
      files.forEach(file => {
        console.log(file)
        const { name, size, type } = file
        // 확장자 체크
        // 갯수 체크
        // 동일이름 체크

        if (campaignType === 'display') {
          if (DISPLAY_TYPE_LIST.includes(type.split('/')[1]) && (this.imageList.length + fileList.length) < LIMIT_COUNT) {
            fileList.push({
              name, size, file
            })
          }
        } else {
          if (BIZ_TYPE_LIST.includes(type.split('/')[1]) && (this.imageList.length + fileList.length) < LIMIT_COUNT) {
            fileList.push({
              name, size, file
            })
          }
        }
      })
      const dialogState = { title: 'DA 업로드 작업', mainText: '파일을 추가할 수 없습니다.', subText: '', type: 'alert' }
      if (existsNotCsv) {
        dialogState.subText = `CSV 파일만 업로드 가능합니다. 재선택 해주세요.`
        this.$dialog(dialogState)
        return
      }
      this.imageList = [...this.imageList, ...fileList]
    },
    selectFile ({ files }) {
      console.log(files[0])
      const file = files[0]
      const dialogState = { title: 'DA 업로드 작업', mainText: '파일을 추가할 수 없습니다.', subText: '', type: 'alert' }
      if (!file.name.endsWith('.csv')) {
        dialogState.subText = `CSV 파일만 업로드 가능합니다. 재선택 해주세요.`
        this.$dialog(dialogState)
        return
      }
      this.uploadFile = {
        name: file.name,
        size: file.size,
        file: file
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.common__list-layout {
  padding: 10px 0;
}
.row-sub-description-text {
  font-size: 12px;
  line-height: 15px;
}
.underline-text {
  color: $pointRed;
  text-decoration: underline;
  margin-left: 5px;
}
</style>
