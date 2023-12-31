
<template>
    <div style="position: relative">
        <svg :style="mirror ? 'transform: scale(-1, 1)' : 'transform: scale(1, 1)'" :width="side + 'px'" :height="side + 'px'" :viewBox="'0 0 ' + side + ' ' + side" ref="_svg"
             @touchmove="handleTouchMove"
             @click="handleClick"
             @mousedown="handleMouseDown"
             @mouseup="handleMouseUp"
        >
            <g>
                <circle :stroke="circleColor" fill="none" :stroke-width="cpMainCircleStrokeWidth" :cx="cpCenter" :cy="cpCenter" :r="radius"></circle>
                <path :stroke="progressColor" fill="none" :stroke-width="cpPathStrokeWidth" :d="cpPathD"></path>
                <circle :fill="knobColor" :r="cpKnobRadius" :cx="cpPathX" :cy="cpPathY"></circle>
            </g>
        </svg>
        <div style="position: absolute; top: 0; display: flex; align-items: center; justify-content: center; z-index: -1" :style="'width:' + side + 'px;' + 'height:' + side + 'px;'">
            <slot></slot>
        </div>
    </div>
</template>
<script>
    import TouchPosition from '../modules/touch_position.js'
    import CircleSliderState from '../modules/circle_slider_state.js'
    import {defineProps, ref, watch, watchEffect, toRefs} from "vue";




    export default {
        name: 'CircleSliderCustom',
        created () {
            this.stepsCount = 1 + (this.max - this.min) / this.stepSize
            this.steps = Array.from({
                length: this.stepsCount
            }, (_, i) => this.min + i * this.stepSize)

            this.circleSliderState = new CircleSliderState(this.steps, this.startAngleOffset, this.value)
            this.angle = this.circleSliderState.angleValue
            this.currentStepValue = this.circleSliderState.currentStep

            let maxCurveWidth = Math.max(this.cpMainCircleStrokeWidth, this.cpPathStrokeWidth)
            this.radius = (this.side / 2) - Math.max(maxCurveWidth, this.cpKnobRadius * 2) / 2
            this.updateFromPropValue(this.value)
        },
        mounted () {
            this.touchPosition = new TouchPosition(this.$refs._svg, this.radius, this.radius / 2, this.$props.mirror)
        },
        props: {
            startAngleOffset: {
                type: Number,
                required: false,
                default: 0
            },
            value: {
                type: Number,
                required: false,
                default: 0
            },
            side: {
                type: Number,
                required: false,
                default: 100
            },
            stepSize: {
                type: Number,
                required: false,
                default: 1
            },
            min: {
                type: Number,
                required: false,
                default: 0
            },
            max: {
                type: Number,
                required: false,
                default: 100
            },
            circleColor: {
                type: String,
                required: false,
                default: '#334860'
            },
            progressColor: {
                type: String,
                required: false,
                default: '#00be7e'
            },
            knobColor: {
                type: String,
                required: false,
                default: '#00be7e'
            },
            knobRadius: {
                type: Number,
                required: false,
                default: null
            },
            knobRadiusRel: {
                type: Number,
                required: false,
                default: 7
            },
            circleWidth: {
                type: Number,
                required: false,
                default: null
            },
            circleWidthRel: {
                type: Number,
                required: false,
                default: 20
            },
            progressWidth: {
                type: Number,
                required: false,
                default: null
            },
            progressWidthRel: {
                type: Number,
                required: false,
                default: 10
            },
            mirror: {
                type: Boolean,
                required: false,
                default: false
            }
            // limitMin: {
            //   type: Number,
            //   required: false,
            //   default: null
            // },
            // limitMax: {
            //   type: Number,
            //   required: false,
            //   default: null
            // }
        },
        data () {
            return {
                steps: null,
                stepsCount: null,
                radius: 0,
                angle: 0,
                currentStepValue: 0,
                mousePressed: false,
                circleSliderState: null,
                mousemoveTicks: 0
            }
        },
        computed: {
            // cpStartAngleOffset () {
            //   if (!this.minStepLimit) {
            //     return 0
            //   }
            // },
            cpCenter () {
                return this.side / 2
            },
            cpAngle () {
                return this.angle + Math.PI / 2
            },
            cpMainCircleStrokeWidth () {
                return this.circleWidth || (this.side / 2) / this.circleWidthRel
            },
            cpPathDirection () {
                return (this.cpAngle < 3 / 2 * Math.PI) ? 0 : 1
            },
            cpPathX () {
                return this.cpCenter + this.radius * Math.cos(this.cpAngle)
            },
            cpPathY () {
                return this.cpCenter + this.radius * Math.sin(this.cpAngle)
            },
            cpPathStrokeWidth () {
                return this.progressWidth || (this.side / 2) / this.progressWidthRel
            },
            cpKnobRadius () {
                return this.knobRadius || (this.side / 2) / this.knobRadiusRel
            },
            cpPathD () {
                let parts = []
                parts.push('M' + this.cpCenter)
                parts.push(this.cpCenter + this.radius)
                parts.push('A')
                parts.push(this.radius)
                parts.push(this.radius)
                parts.push(0)
                parts.push(this.cpPathDirection)
                parts.push(1)
                parts.push(this.cpPathX)
                parts.push(this.cpPathY)
                return parts.join(' ')
            }
        },
        methods: {
            /*
             */
            fitToStep (val) {
                return Math.round(val / this.stepSize) * this.stepSize
            },

            /*
             */
            handleClick (e) {
                this.touchPosition.setNewPosition(e)
                if (this.touchPosition.isTouchWithinSliderRange) {
                    const newAngle = this.touchPosition.sliderAngle
                    this.animateSlider(this.angle, newAngle)
                }
            },

            /*
             */
            handleMouseDown (e) {
                e.preventDefault()
                this.mousePressed = true
                window.addEventListener('mousemove', this.handleWindowMouseMove)
                window.addEventListener('mouseup', this.handleMouseUp)
            },

            /*
             */
            handleMouseUp (e) {
                e.preventDefault()
                this.mousePressed = false
                window.removeEventListener('mousemove', this.handleWindowMouseMove)
                window.removeEventListener('mouseup', this.handleMouseUp)
                this.mousemoveTicks = 0
            },

            /*
             */
            handleWindowMouseMove (e) {
                e.preventDefault()
                if (this.mousemoveTicks < 5) {
                    this.mousemoveTicks++
                    return
                }

                this.touchPosition.setNewPosition(e)
                this.updateSlider()
            },

            /*
             */
            handleTouchMove (e) {
                this.$emit('touchmove')
                // Do nothing if two or more fingers used
                if (e.targetTouches.length > 1 || e.changedTouches.length > 1 || e.touches.length > 1) {
                    return true
                }

                const lastTouch = e.targetTouches.item(e.targetTouches.length - 1)
                this.touchPosition.setNewPosition(lastTouch)

                if (this.touchPosition.isTouchWithinSliderRange) {
                    e.preventDefault()
                    this.updateSlider()
                }
            },

            /*
             */
            updateAngle (angle) {
                this.circleSliderState.updateCurrentStepFromAngle(angle)
                this.angle = this.circleSliderState.angleValue
                this.currentStepValue = this.circleSliderState.currentStep
                this.$emit('update:value', this.currentStepValue)
            },

            updateAngleFromPropValue (angle) {
                this.circleSliderState.updateCurrentStepFromAngle(angle)
                this.angle = this.circleSliderState.angleValue
                this.currentStepValue = this.circleSliderState.currentStep
                // this.$emit('update:value', this.currentStepValue)
            },

            /*
             */
            updateFromPropValue (value) {
                let stepValue = this.fitToStep(value)
                this.circleSliderState.updateCurrentStepFromValue(stepValue)
                this.animateSliderFromPropValue(this.angle, this.circleSliderState.angleValue)
                this.angle = this.circleSliderState.angleValue

                this.currentStepValue = stepValue
                // this.$emit('update:value', this.currentStepValue)
            },

            /*
             */
            updateSlider () {
                const angle = this.touchPosition.sliderAngle
                if (Math.abs(angle - this.angle) < Math.PI) {
                    this.updateAngle(angle)
                }
            },

            /*
             */
            animateSlider (startAngle, endAngle) {
                const direction = startAngle < endAngle ? 1 : -1
                const curveAngleMovementUnit = direction * this.circleSliderState.angleUnit * 2

                const animate = () => {
                    if (Math.abs(endAngle - startAngle) < Math.abs(2 * curveAngleMovementUnit)) {
                        this.updateAngle(endAngle)
                    } else {
                        const newAngle = startAngle + curveAngleMovementUnit
                        this.updateAngle(newAngle)
                        this.animateSlider(newAngle, endAngle)
                    }
                }

                window.requestAnimationFrame(animate)
            },
            animateSliderFromPropValue (startAngle, endAngle) {
                const direction = startAngle < endAngle ? 1 : -1
                const curveAngleMovementUnit = direction * this.circleSliderState.angleUnit * 2

                const animate = () => {
                    if (Math.abs(endAngle - startAngle) < Math.abs(2 * curveAngleMovementUnit)) {
                        this.updateAngleFromPropValue(endAngle)
                    } else {
                        const newAngle = startAngle + curveAngleMovementUnit
                        this.updateAngleFromPropValue(newAngle)
                        this.animateSliderFromPropValue(newAngle, endAngle)
                    }
                }

                window.requestAnimationFrame(animate)
            }
        },
        watch: {
            value: {
                handler(newValue, oldValue) {
                    if(typeof newValue === 'string') newValue = 0
                    this.updateFromPropValue(newValue)
                }
            }
        }
    }
</script>
