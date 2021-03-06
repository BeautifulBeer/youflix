<template>
    <v-container
        v-if="visible"
        id="movie-detail"
        fluid
        px-5
        ma-0
        style="background-color: black; height: 100vh; padding-top: 10vh;"
    >
        <AnimateWhenVisible name="fade">
            <v-row
                justify="start"
            >
                <v-col
                    sm="12"
                    md="3"
                    order="1"
                >
                    <div class="movie-detail-wrapper">
                        <div class="detail-title">
                            <span>{{ pmovie.title }}</span>
                        </div>
                        <div class="detail-rating">
                            <AnimateWhenVisible name="fade">
                                <div class="wrapper">
                                    <span class="label">
                                        예상별점
                                    </span>
                                    <span class="score">
                                        {{ predictedScore | getPredictionScore }}
                                    </span>
                                </div>
                                <div class="wrapper">
                                    <span class="label">
                                        영화별점
                                    </span>
                                    <span class="score">
                                        {{ pmovie.vote_average | scoreConverter }}
                                    </span>
                                </div>
                            </AnimateWhenVisible>
                        </div>
                        <div
                            class="detail-content"
                        >
                            <span>{{ pmovie.overview }}</span>
                        </div>
                        <div class="detail-inform">
                            <AnimateWhenVisible name="fade">
                                <span>{{ pmovie.runtime | runTimeLabel }}</span>
                            </AnimateWhenVisible>
                        </div>
                        <div class="detail-play">
                            <div class="detail-play-content">
                                <span>예고편 보기</span>
                                <i class="material-icons">play_circle_outline</i>
                            </div>
                        </div>
                        <div>
                            <span
                                class="taste-word"
                                style="color: white;"
                            >
                                {{ ratingWord }}
                            </span>
                            <div style="padding-top: 7px;">
                                <span @click="ratingMovie()">
                                    <v-rating
                                        id="ratingStar"
                                        v-model="rating"
                                        dense
                                        color="white"
                                        background-color="white"
                                        half-increments
                                        hover
                                    />
                                </span>
                            </div>
                        </div>
                    </div>
                </v-col>
                <v-col
                    style="position: relative; padding: 0;"
                    sm="12"
                    md="9"
                    order="2"
                >
                    <div class="shadow" />
                    <a href="#personalized-section">
                        <i
                            class="material-icons closebtn"
                            @click="close"
                        >close</i>
                    </a>
                    <v-img
                        class="detail-img"
                        contain
                        :src="pmovie.backdrop_path | imagePath"
                    />
                </v-col>
            </v-row>
        </AnimateWhenVisible>
    </v-container>
</template>

<script>
import { createNamespacedHelpers } from 'vuex';

const userMapState = createNamespacedHelpers('users').mapState;
const ratingMapActions = createNamespacedHelpers('ratings').mapActions;
const movieMapActions = createNamespacedHelpers('movies').mapActions;
const movieMutations = createNamespacedHelpers('movies').mapMutations;

export default {
    filters: {
        runTimeLabel(value) {
            return ['상영시간', ':', value, '분'].join(' ');
        },
        imagePath(value) {
            return value === '' ? '/static/img/commingsoon.jpg' : value;
        },
        scoreConverter(value) {
            return (value / 2).toFixed(1);
        },
        getPredictionScore(value) {
            if (value === 0) {
                return '-';
            }
            return value.toFixed(1);
        }
    },
    props: {
        visible: {
            type: Boolean,
            default: false
        },
        pmovie: {
            type: Object,
            default: null
        },
        close: {
            type: Function,
            default: null
        },
        rating: {
            type: Number,
            default: 0
        },
        ratingWord: {
            type: String,
            default: '이미 본 작품인가요?'
        }
    },
    data() {
        return {
            predictedScore: 0
        };
    },
    computed: {
        ...userMapState(['user'])
    },
    watch: {
        // eslint-disable-next-line
        pmovie: function(val) {
            this.getPrediction([this.user.email, this.pmovie.id]).then((score) => {
                if (score !== -1) {
                    this.predictedScore = score;
                }
                this.$forceUpdate();
            });
        },
        // eslint-disable-next-line
        visible: function(val) {
            this.$log.debug('MovieDetail.vue visible watcher', val);
            if (val) {
                this.disableScroll();
                this.setHeaderVisible(!val);
            } else {
                this.enableScroll();
                this.setHeaderVisible(!val);
            }
        }
    },
    methods: {
        ...ratingMapActions(['rateMovie']),
        ...movieMapActions(['getPrediction']),
        ...movieMutations(['setHeaderVisible']),
        ratingMovie() {
            this.rateMovie(
                {
                    email: this.user.email,
                    movie_id: this.pmovie.id,
                    ratingValue: this.rating
                }
            ).then((ret) => {
                this.ratingWord = ret;
            });
        },
        disableScroll() {
            const sliderContainer = document.getElementById('slider-container');
            this.$log.debug('MovieDetail.vue disableScroll sliderContainer', sliderContainer);
            if (sliderContainer) {
                let { left, bottom } = sliderContainer.getBoundingClientRect();
                left += window.scrollX;
                bottom += window.scrollY - 40;
                if (left < 0) {
                    left = 0;
                }
                this.$log.debug('MovieDetail.vue disableScroll position', left, bottom);
                window.scrollTo(left, bottom);
                window.onscroll = () => {
                    window.scrollTo(left, bottom);
                };
            }
        },
        enableScroll() {
            window.onscroll = () => {};
        }
    }
};
</script>

<style lang="scss" scoped>

.detail-img{
    width: 100%;
    height: 100%;
    z-index: 5;
}

.shadow {
    z-index: 10;
    width: 100%;
    height: 100%;
    display: inline-block;
    position: absolute;
    -moz-box-shadow: rgba(0, 0, 0, 1) 150px 150px 150px inset;
    -webkit-box-shadow: rgba(0, 0, 0, 1) 150px 150px 150px inset;
    box-shadow: rgba(0, 0, 0, 1) 150px 150px 150px inset;
}

.closebtn{
    z-index: 15;
    position: absolute;
    right: 50px;
    font-size: 3em;
    color: gray;

    &:hover{
        color: white;
        cursor: pointer;
    }
}


.movie-detail-wrapper{
    margin-top: 50px;
    padding: 50px;
    position: relative;
    width: 150%;
    height: 100%;
    z-index: 15;

    span{
        color: gray;
    }
}


.detail-title {
    margin: 5px;
    span{
        color: white;
        font-size: 3em;
        font-weight: bold;
    }
}

.detail-rating {
    height: 40px;
    margin: 10px;
    font-size: 1.2em;
    font-weight: bold;
    margin-bottom: 30px;
    .wrapper{
        width: 130px;
        height: 40px;
        position: relative;
        background-color: white;
        border: 3px solid white;
        margin-right: 20px;
        float: left;
        .label{
            background-color: black;
            position:absolute;
            padding-top: 5px;
            z-index: 25;
            left: 0%;
            width: 70%;
            height: 100%;
            top: 50%;
            color: white;
            transform: translateY(-50%);
            text-align: center;
            vertical-align: middle;
        }
        .score{
            background-color: white;
            position:absolute;
            z-index: 20;
            width: 25%;
            top: 50%;
            right: 0%;
            color: black;
            text-align: center;
            transform: translateY(-50%);
        }
    }
}

.detail-play{
    position: relative;
    min-height: 50px;
    margin: 10px;
}

.detail-play-content{
    position: relative;
    min-height: 60px;
    max-width: 200px;
    box-sizing: border-box;
    background-color: #e50914;

    span {
        color: white;
        position: absolute;
        top: 50%;
        right: 3.2em;
        font-size: 1.3em;
        transform: translateY(-50%);
    };
    i {
        position: absolute;
        min-height: 50px;
        font-size: 3em;
        right: 0.2em;
        top: 50%;
        transform: translateY(-50%);
    };
}

.detail-inform{
    font-size: 1.2em;
    margin: 10px;
    margin-bottom: 50px;
}

.detail-content{
    font-size: 1em;
    margin: 10px;
    line-height: 160%;
    // max-height: 300px;
    margin-bottom: 30px;

    &:first-letter{
        font-size: 150%;
        text-transform: uppercase;
    }
}

.taste-word {

    font-size: 1.0em;
    float: left;
    margin-top: 10px;
    margin-right: 20px;
    margin-left: 10px;
}

</style>
