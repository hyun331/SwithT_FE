<template>
    <LectureDetailInfoComponent 
        :lectureId=this.lectureId
        />
    <v-container width="50%"  style="margin-top: 60px;">
        <h4  class="div-title">{{this.title}}  <br>신청 리스트  </h4>
        <br>
        <v-row v-for="apply in lessonApplyList" :key="apply.id" >
            <v-col>
                <v-card class="custom-border">
                    <v-card-text class="d-flex justify-space-between align-center">
                        <div>
                            <v-avatar class="profile-image" style="height: 50px; width: 50px; ">
                                <v-img :src="apply.tuteeProfileImage" alt="튜티 프로필"/>
                              </v-avatar>
                            
                            <span style="font-size: 20px; margin:20px;">{{ apply.tuteeName }}</span>
                            <v-icon @click="clickChatRoom(apply.chatRoomId, apply.memberId)">mdi-message-outline</v-icon>
                        </div>


                        <div>
                            <span class="ml-3 font-weight-class" style="color: blue; font-weight:bold; font-size: 18px;" v-if="apply.status === 'WAITING'">결제 대기중</span>
                            
                            <v-btn color="#3232FF" class="ml-3 font-weight-class" v-else
                                @click="clickApplyRejectBtn('apply', apply.applyId, apply.tuteeName)">수락</v-btn>
                            <v-btn color="#f03e3e" class="ml-3 font-weight-class"
                                @click="clickApplyRejectBtn('reject', apply.applyId, apply.tuteeName)">거절</v-btn>
                        </div>


                    </v-card-text>
                </v-card>
            </v-col>
        </v-row>

        <v-pagination v-model="frontendPage" :length="totalPages" @click="handlePageChange"></v-pagination>

        <YesOrNoModal
        v-model:dialog="yesOrNoModal"
        @confirmed="handleYesBtn"
        @update:dialog="yesOrNoModal = $event"
        :icon=this.icon
        :title=this.modalTitle
        :contents=this.modalContents
        yesBtnName="확인"
        />

        <AlertModal
        v-model="alertModal" 
        @update:dialog="alertModal = $event"
        icon=mdi-alert-circle-outline
        :title=this.alertModalTtile
        :contents=this.alertModalContents
        />

    </v-container>
</template>

<script>
import YesOrNoModal from '@/components/YesOrNoModal.vue';
import LectureDetailInfoComponent from '@/components/LectureDetailInfoComponent.vue';
import AlertModal from '@/components/AlertModal.vue';
import axios from 'axios';
export default {
    components:{
        YesOrNoModal,
        LectureDetailInfoComponent,
        AlertModal,
    },

    data() {
        return {
            lectureId: this.$route.query.lectureId,
            lessonApplyList: [],
            page: 0,
            size: 5,
            totalPages: 0,
            frontendPage: 1,
            title: this.$route.query.title,
            lectureGroupId: null,
            selectedApplyId: null,

            yesOrNoModal: false,
            modalTitle: 'kk',
            modalContents: 'kk',

            alertModal: false,
            alertModalTtile: '',
            alertModalContents: '',

        };

    },

    created() {
        this.showApplyList();
    },

    methods: {
        async showApplyList() {
            try {
                let params = {
                    size: this.size,
                    page: this.page
                };
                this.lectureGroupId = this.$route.query.lectureGroupId;
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/lecture-service/single-lecture-apply-list/${this.lectureGroupId}`, { params });
                this.lessonApplyList = response.data.result.content;
                this.totalPages = response.data.result.totalPages;
            } catch (e) {
                console.log(e.response.data.error_message);
            }
        },

        handlePageChange() {
            this.page = this.frontendPage - 1;
            this.showApplyList();
        },


        clickApplyRejectBtn(applyOrReject, applyId, tuteeName){
            if(applyOrReject == 'apply'){
                this.modalTitle = '결제 요청';
                this.modalContents = tuteeName+'님께 결제요청을 보내시겠습니까?';
                this.icon = "mdi-credit-card-fast-outline";
            }else{
                this.modalTitle = '신청 거절';
                this.modalContents = tuteeName+'님의 신청을 거절하시겠습니까?';
                this.icon = "mdi-alert-circle-outline";
            }
            this.selectedApplyId = applyId;
            this.yesOrNoModal = true;


        },

        handleYesBtn(){
            if(this.modalTitle == '결제 요청'){
                this.clickApplyAdmit(this.selectedApplyId);
            }else{
                this.clickApplyReject(this.selectedApplyId);
            }
            
            this.yesOrNoModal = false;

        },

        async clickApplyAdmit(applyId) {
            try {
                const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/lecture-service/single-lecture-payment-request/${applyId}`);
                console.log(response.data.result);
                window.location.reload();
            } catch (e) {
                this.alertModalTtile = "결제 요청 실패";
                this.alertModalContents =  "한번에 한 튜티에게 결제 요청을 보낼 수 있습니다.";
                this.alertModal = true;

            }
        },

        async clickApplyReject(applyId) {
            try {
                const response = await axios.patch(`${process.env.VUE_APP_API_BASE_URL}/lecture-service/single-lecture-apply-reject/${applyId}`);
                console.log(response.data.result);
                window.location.reload();
            } catch (e) {
                console.log(e.response.data);
                alert("신청 취소에 실패했습니다.");
            }

        },

        async clickChatRoom(chatRoomId, tuteeId){
            console.log(chatRoomId, tuteeId);
            if(chatRoomId === null){
                //채팅방 생성
                const registerData = {
                    lectureGroupId: this.lectureGroupId,
                    tuteeId: tuteeId
                }
                const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/lecture-service/tutor-lesson-chat-check-or-create`, registerData);
                console.log(response.data.result);
                chatRoomId = response.data.result.roomId;
                
            }
            this.$router.push(`/chat-room?chatRoomId=${chatRoomId}`);
        }

    }
}
</script>
<style scoped>
.custom-border {
    border: 2px solid #cbd1ff;
    border-radius: 8px;
    box-shadow: none !important;
}

div,
.font-weight-class {
    font-weight: 900;
}

.submit-group {
    background-color: #0d6efd;
    color: #f5f5f5;
    width: 100px;
    height: 40px;
    line-height: 40px;
    border-radius: 10px;
    right: 0;
    font-weight: bold;
  }
.submit-group:hover {
    cursor: pointer;
}

.div-title{
    font-weight:bold;
    height: 100px;
    border: 2px solid #cbd1ff;
    border-radius: 8px;
    align-content: center;
}
</style>