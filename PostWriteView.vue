<template>
  <div class="px-10 py-5" style=" margin-bottom: 60px;">

    <div class="card">
      <div class="card-body">
        <div class="mb-10">
          <label class="form-lable fw-bold">제목</label>
          <input type="text" class="form-control" v-model="postTitle" placeholder="제목 입력">
        </div>


        <div class="mb-10">
          <label class="form-lable fw-bold">내용</label>
          <input type="text" class="form-control" rows="3" v-model="postContent" placeholder="내용 입력">
        </div>
        <div class="mb-10">
          <label class="form-lable fw-bold">카테고리</label>
          <input type="text" class="form-control" rows="3" v-model="postCategory" placeholder="카테고리 입력">
        </div>
        <div class="mb-10">
          <label class="form-lable fw-bold">생성 날짜</label>
          <input type="text" class="form-control" rows="3" v-model="postCreateDate" placeholder="생성 입력">
        </div>
        <div class="mb-10">
          <label class="form-lable fw-bold">좋아요 수</label>
          <input type="text" class="form-control" rows="3" v-model="postLikes" placeholder="좋아요 입력">
        </div>
        <div class="mb-10">
          <label class="form-lable fw-bold">댓글 수</label>
          <input type="text" class="form-control" rows="3" v-model="postComments" placeholder="댓글 입력">
        </div>

        <div class="mt-4">
          <input type="file" id="uploadImage" hidden @change="getFilename($event.target.files)">
          <label for="uploadImage" class="d-flex justify-content-center">
            <img src="/assets/media/avatars/thumb_ing.png" id="preview" width="50%">
          </label>
          <label>이미지 경로</label>
          <input type="hidden" v-model="postThumbnail"></input>
        </div>
      </div>

      <div class="card-footer">

        <div class="d-flex justify-content-around ">
          <button class="btn btn-light-primary w-45 px-20 py-5" @click="addPost()">
            <span class="fs-3 fw-bold">저장</span>
          </button>
          <button class="btn btn-light-secondary w-45  px-20 py-5" @click="goToPost()">
            <span class="fs-3 fw-bold">취소</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

//라우터
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
// 스토어 불러오기
import { storeToRefs } from 'pinia'

import { useAppStore } from '@/stores/app'
const appStore = useAppStore();
const { title } = storeToRefs(appStore);

import { usePostStore } from '@/stores/post'
const postStore = usePostStore();
const { posts, mode, selectedIndex } = storeToRefs(postStore);

// 업로드
import { useUpload } from '@/util/upload.js';
const { upload } = useUpload()

import { requestConfig } from '../../app.config';

const postTitle = ref('');
const postContent = ref('');
const postCategory = ref('');
const postCreateDate = ref('');
const postLikes = ref('');
const postComments = ref('');
const postThumbnail = ref('');
//게시물 목록을 위한 변수

// 선택된 파일
const selectedFile = ref('')

onMounted(() => {
  console.log(`DocumentView::onMounted호출됨`);
  title.value = '게시물 추가'
  // 수정 모드인 경우, 입력 상자에 선택된 아이템의 값 넣어주기
  if (mode.value == 'modify') {
    const selected = posts.value[selectedIndex.value]
    postTitle.value = selected.title
    postContent.value = selected.content
    postCategory.value = selected.category
    postCreateDate.value = selected.createDate
    postLikes.value = selected.likes
    postComments.value = selected.comments
    postThumbnail.value = selected.thumbnail
    title.value = '게시물 수정'
  }
})

async function getFilename(files) {
  selectedFile.value = files[0]
  await base24()
}

function base24(){
  return new Promise((resolve, reject) =>{
      let reader = new FileReader()
    reader.onload = e => {
      resolve(e.target.result)

      const previewImage = document.querySelector('#preview')
      previewImage.src = e.target.result
    }

    reader.readAsDataURL(selectedFile.value)
  })
}

async function addPost() {
  console.log(`addPost 호출됨`);


  const title = postTitle.value
  const content = postContent.value
  const category = postCategory.value
  const createDate = postCreateDate.value
  const likes = postLikes.value
  const comments = postComments.value
  const thumbnail = postThumbnail.value

  const item = {
    // id: String(animals.length + 1),
    title: title,
    content: content,
    category: category,
    createDate: createDate,
    likes: likes,
    comments: comments,
    thumbnail: thumbnail

  }

  if (mode.value == 'add') {
    requestPostAdd(item)

  } else if (mode.value == 'modify') {

    item.id = posts.value[selectedIndex.value].id
    requestPostModify(item)

  }

}

async function requestPostAdd(item) {
  try {

    let response = await upload(selectedFile.value, (progress) => {
      console.log(`업로드 진행률 ㅣ ${progress}`)
    })

    console.log(`업로드 응답 -> ${JSON.stringify(response)}`)

    item.thumbnail = `${requestConfig.baseUrl}${response.data.filename}`

    response = await axios({
      method: 'post',
      baseURL: 'http://localhost:8001',
      url: 'post/v1/add',
      data: item,
      timeout: 5000,
      responseType: 'json'
    })

    console.log(`응답 -> ${JSON.stringify(response.data)}`);
    goToPost()

  } catch (err) {
    console.error(`에러->${err}`);

  }
}

async function requestPostModify(item) {
  try {

    let response = await upload(selectedFile.value, (progress) => {
      console.log(`업로드 진행률 ㅣ ${progress}`)
    })

    console.log(`업로드 응답 -> ${JSON.stringify(response)}`)

    item.thumbnail = `${requestConfig.baseUrl}${response.data.filename}`

     response = await axios({
      method: 'post',
      baseURL: 'http://localhost:8001',
      url: 'post/v1/modify',
      data: item,
      timeout: 5000,
      responseType: 'json'
    })
   

    console.log(`응답 -> ${JSON.stringify(response.data)}`);
    goToPost()

  } catch (err) {
    console.error(`에러->${err}`);

  }
}
// 게시글목록화면
function goToPost() {


  router.push('/document');
}


</script>
<style scoped></style>
