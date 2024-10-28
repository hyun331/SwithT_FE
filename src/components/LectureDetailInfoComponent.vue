<template>
    <section class="top-info">
        <v-row  class="align-center">
            <v-col cols="7">
                <div>
                    <span class="lectureType">
                        {{ formattedLectureType }}
                    </span>
                    <span class="category" style="font-size: 18px;">
                        {{ formattedCategory }}
                    </span>
                </div>
                <div class="title">{{ lectureInfo?.title }}</div>
                <div class="memberName"> {{ lectureInfo?.memberName }} 튜터</div>
                <div class="avgScore">
                    <span class="mdi mdi-star" style="font-size: 18px;"> </span>
                    <span style="font-size: 18px; margin-left: 3px;"> {{ lectureInfo?.avgScore }}점</span>
                </div>
            </v-col>
            <v-col cols="5" class="d-flex align-center justify-center">
                <div class="thumbnail">
                    <img :src="lectureInfo?.image" alt="Lecture Thumbnail"/>
                </div>
            </v-col>
        </v-row>
    </section>
</template>

<script>
import axios from 'axios';

export default {
    props:['lectureId'],
    data() {
        return {
            lectureInfo: null, // 강의 정보를 저장할 객체
        };
    },
    computed: {
        formattedLectureType(){
            switch(this.lectureInfo?.lectureType) {
                case 'LECTURE':
                    return '강의';
                case 'LESSON':
                    return '과외';
                default:
                    return this.lectureInfo?.lectureType;
            }
        },
        formattedCategory() {
            switch (this.lectureInfo?.category) {
                case 'CAREER':
                    return '취업&직무';
                case 'HOBBY':
                    return '취미';
                case 'ADMISSION':
                    return '입시';
                case 'DEVELOPMENT':
                    return '자기계발';
                default:
                    return this.lectureInfo?.category;
            }
        },
    },
    mounted() {
        this.fetchLectureDetail();
    },
    methods: {
        async fetchLectureDetail() {
            // const lectureId = this.$route.params.id; // URL에서 강의 ID 가져오기
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/lecture-service/lecture-detail/${this.lectureId}`);
                this.lectureInfo = response.data.result; // 가져온 강의 정보를 저장
                console.log(this.lectureInfo);
            } catch (error) {
                console.error('강의 정보를 가져오는 데 실패했습니다:', error);
            }
        },
    },
};
</script>

<style scoped>
.v-container {
    padding: 0 !important;
}
.top-info {
    background-color: #121212;
    color: white;
    padding: 100px 20vw 40px;
    text-align: left; 
}
.lectureType {
    display: inline-block;
    margin-right: 5px;
    background-color: #6C97FD;
    width: 50px;
    border-radius: 5px;
    font-size: 18px;
    font-weight: bold;
    text-align: center;
}
.title {
    font-size: 40px;
    font-weight: bold;
}
.memberName {
    font-size: 20px;
}
.avgScore {
    font-size: 1.5rem;
}
.thumbnail img {
    border-radius: 8px;
    height: 300px;
    object-fit: contain;
}
</style>
