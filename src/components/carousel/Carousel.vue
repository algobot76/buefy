<template>
    <div
        class="carousel"
        :class="{'is-overlay': overlay}"
        @mouseenter="pauseTimer"
        @mouseleave="startTimer">
        <progress
            v-if="progress"
            class="progress"
            :class="progressType"
            :value="activeItem"
            :max="carouselItems.length - 1">
            {{ carouselItems.length - 1 }}
        </progress>
        <div
            class="carousel-list"
            @mousedown="dragStart"
            @mouseup="dragEnd"
            @touchstart.stop="dragStart"
            @touchend.stop="dragEnd">
            <slot />
            <div
                v-if="arrow"
                class="carousel-arrow"
                :class="{'is-hovered': arrowHover}">
                <b-icon
                    v-if="checkArrow(0)"
                    class="has-icons-left"
                    @click.native.prevent="prev"
                    :pack="iconPack"
                    :icon="iconPrev"
                    :size="iconSize"
                    both />
                <b-icon
                    v-if="checkArrow(carouselItems.length - 1)"
                    class="has-icons-right"
                    @click.native.prevent="next"
                    :pack="iconPack"
                    :icon="iconNext"
                    :size="iconSize"
                    both />
            </div>
        </div>
        <div
            v-if="autoplay && pauseHover && pauseInfo && isPause"
            class="carousel-pause">
            <span class="tag">Pause</span>
        </div>
        <div
            v-if="indicator"
            class="carousel-indicator"
            :class="indicatorClasses">
            <a
                v-for="(item, index) in carouselItems"
                class="indicator-item"
                :class="{'is-active': index === activeItem}"
                @mouseover="modeChange('hover', index)"
                @click="modeChange('click', index)"
                :key="index">
                <slot
                    :i="index"
                    name="indicators">
                    <span class="indicator-style" :class="indicatorStyle"/>
                </slot>
            </a>
        </div>
    </div>
</template>

<script>
import config from '../../utils/config'

import Icon from '../icon/Icon'

export default {
    name: 'BCarousel',
    components: {
        [Icon.name]: Icon
    },
    props: {
        value: {
            type: Number,
            default: 0
        },
        animated: {
            type: String,
            default: 'slide'
        },
        interval: Number,
        hasDrag: {
            type: Boolean,
            default: true
        },
        autoplay: {
            type: Boolean,
            default: true
        },
        pauseHover: {
            type: Boolean,
            default: true
        },
        pauseInfo: {
            type: Boolean,
            default: true
        },
        arrow: {
            type: Boolean,
            default: true
        },
        arrowBoth: {
            type: Boolean,
            default: true
        },
        arrowHover: {
            type: Boolean,
            default: true
        },
        iconPack: String,
        iconSize: String,
        iconPrev: {
            type: String,
            default: config.defaultIconPrev
        },
        iconNext: {
            type: String,
            default: config.defaultIconNext
        },
        indicator: {
            type: Boolean,
            default: true
        },
        indicatorBackground: Boolean,
        indicatorCustom: Boolean,
        indicatorCustomSize: {
            type: String,
            default: 'is-small'
        },
        indicatorInside: {
            type: Boolean,
            default: true
        },
        indicatorMode: {
            type: String,
            default: 'click'
        },
        indicatorPosition: {
            type: String,
            default: 'is-bottom'
        },
        indicatorStyle: {
            type: String,
            default: 'is-dots'
        },
        overlay: Boolean,
        progress: Boolean,
        progressType: {
            type: String,
            default: 'is-primary'
        }
    },
    data() {
        return {
            _isCarousel: true,
            activeItem: this.value,
            carouselItems: [],
            isPause: false,
            dragX: 0,
            timer: null
        }
    },
    computed: {
        indicatorClasses() {
            return [
                {
                    'has-background': this.indicatorBackground,
                    'has-custom': this.indicatorCustom,
                    'is-inside': this.indicatorInside
                },
                this.indicatorCustom && this.indicatorCustomSize,
                this.indicatorInside && this.indicatorPosition
            ]
        }
    },
    watch: {
        /**
        * When v-model is changed set the new active tab.
        */
        value(value) {
            this.changeItem(value, false)
        },
        /**
        * When tab-items are updated, set active one.
        */
        carouselItems() {
            if (this.activeItem < this.carouselItems.length) {
                this.carouselItems[this.activeItem].isActive = true
            }
        },
        /**
         *  When autoplay is change, set by status
         */
        autoplay(status) {
            status ? this.startTimer() : this.pauseTimer()
        }
    },
    methods: {
        startTimer() {
            if (!this.autoplay || this.timer) return
            this.isPause = false
            this.timer = setInterval(() => {
                this.next()
            }, (this.interval || config.defaultCarouselInterval))
        },
        pauseTimer() {
            if (!this.pauseHover && this.autoplay) return
            this.isPause = true
            if (this.timer) {
                clearInterval(this.timer)
                this.timer = null
            }
        },
        /**
        * Change the active item and emit change event.
        * action only for animated slide, there true = next, false = prev
        */
        changeItem(newIndex, action) {
            if (this.activeItem === newIndex) return
            this.carouselItems[this.activeItem].status(false, action)
            this.carouselItems[newIndex].status(true, action)
            this.activeItem = newIndex
            this.$emit('change', newIndex)
        },
        // Indicator trigger, emit input event and change active item.
        modeChange(trigger, value) {
            if (this.indicatorMode === trigger) {
                this.$emit('input', value)
                this.changeItem(value, false)
            }
        },
        prev() {
            return this.activeItem === 0
                ? this.changeItem(this.carouselItems.length - 1, true)
                : this.changeItem(this.activeItem - 1, true)
        },
        next() {
            return this.activeItem === this.carouselItems.length - 1
                ? this.changeItem(0, false)
                : this.changeItem(this.activeItem + 1, false)
        },
        // checking arrow between both
        checkArrow(value) {
            if (this.arrowBoth) return true
            if (this.activeItem !== value) return true
        },
        // handle drag event
        dragStart(event) {
            if (!this.hasDrag) return
            this.dragx = event.touches ? event.changedTouches[0].pageX : event.pageX
            if (event.touches) {
                this.pauseTimer()
            } else {
                event.preventDefault()
            }
        },
        dragEnd(event) {
            if (!this.hasDrag) return
            const detected = event.touches ? event.changedTouches[0].pageX : event.pageX
            const diffX = detected - this.dragx
            if (Math.abs(diffX) > 50) {
                if (diffX < 0) {
                    this.next()
                } else {
                    this.prev()
                }
            }
            if (event.touches) {
                this.startTimer()
            }
        }
    },
    mounted() {
        if (this.activeItem < this.carouselItems.length) {
            this.carouselItems[this.activeItem].isActive = true
        }
        this.startTimer()
    },
    beforeDestroy() {
        this.pauseTimer()
    }
}
</script>
