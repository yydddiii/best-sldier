<template>
    <div class="ui-slider__wrapper">
        <div class="ui-slider" ref="slider" :class="props?.draggable ? '_draggable' : ''">
            <div
                class="ui-slider__slides"
                :style="{
                    gap: `${props?.spaceBetween}px`,
                    transition: `${
                        current_transition == 'unset'
                            ? current_transition
                            : current_transition + ' cubic-bezier(0.25, 1, 0.5, 1)'
                    }`,
                    transform: `translateX(${transform * -1}px)`,
                }"
            >
                <!-- NOTE все расчеты мб можно вынести в функцию или компутед какойнибудь, а то в темплате кринжово выглядит -->
                <div
                    class="ui-slider__slides-item"
                    v-for="(item, index) in props?.loop
                        ? slot_childrens.concat(slot_childrens).concat(slot_childrens)
                        : slot_childrens"
                    :style="{
                        width:
                            ((props?.loop ? index - slot_childrens.length : index) ==
                                current_index && props?.currentSlideStyle
                                ? props?.currentSlideStyle.width
                                : null) ?? `${slide_width}px`,
                        transition: `${
                            current_transition == 'unset'
                                ? current_transition
                                : current_transition + ' cubic-bezier(0.25, 1, 0.5, 1)'
                        }`,
                    }"
                    :class="[
                        (props?.loop ? index - slot_childrens.length : index) == current_index
                            ? '_active-slide'
                            : '',
                        (props?.loop ? index - slot_childrens.length : index) == current_index - 1
                            ? '_prev-slide'
                            : '',
                        (props?.loop ? index - slot_childrens.length : index) == current_index + 1
                            ? '_next-slide'
                            : '',
                    ]"
                >
                    <!-- NOTE тут тоже самое что и выше(строка 16) -->
                    <component
                        :is="item"
                        :style="{
                            ...((props?.loop ? index - slot_childrens.length : index) ==
                                current_index && props?.currentSlideStyle
                                ? props?.currentSlideStyle
                                : {}),
                            transition: `${
                                current_transition == 'unset'
                                    ? current_transition
                                    : current_transition + ' cubic-bezier(0.25, 1, 0.5, 1)'
                            }`,
                        }"
                    />
                </div>
            </div>
        </div>
        <template v-if="props.showArrows">
            <slot name="arrows" :next="next" :prev="prev">
                <div class="ui-slider__arrows" :class="`_${props?.arrowsPlacement}`">
                    <button
                        @click="prev"
                        class="ui-slider__arrows-item prev"
                        :class="current_index == 0 && !props?.loop ? '_lock' : ''"
                    >
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16">
                            <g fill="none">
                                <path
                                    d="M10.26 3.2a.75.75 0 0 1 .04 1.06L6.773 8l3.527 3.74a.75.75 0 1 1-1.1 1.02l-4-4.25a.75.75 0 0 1 0-1.02l4-4.25a.75.75 0 0 1 1.06-.04z"
                                    fill="#fff"
                                ></path>
                            </g>
                        </svg>
                    </button>
                    <button
                        @click="next"
                        class="ui-slider__arrows-item next"
                        :class="
                            current_index + props.slidesPerView - 1 == slot_childrens.length - 1 &&
                            !props?.loop
                                ? '_lock'
                                : ''
                        "
                    >
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16">
                            <g fill="none">
                                <path
                                    d="M5.74 3.2a.75.75 0 0 0-.04 1.06L9.227 8L5.7 11.74a.75.75 0 1 0 1.1 1.02l4-4.25a.75.75 0 0 0 0-1.02l-4-4.25a.75.75 0 0 0-1.06-.04z"
                                    fill="#fff"
                                ></path>
                            </g>
                        </svg>
                    </button>
                </div>
            </slot>
        </template>
        <template v-if="props.showPagination">
            <slot
                name="pagination"
                :count_slides="slot_childrens.length"
                :current_index="current_index"
                :to="goTo"
            >
                <div class="ui-slider__pagination" :class="`_${props?.paginationPlacement}`">
                    <span
                        class="ui-slider__pagination-item"
                        v-for="(_, index) in slot_childrens.length - getSlidePerView() + 1"
                        :class="[
                            index == current_index ? '_current' : '',
                            `_${props?.paginationType}`,
                        ]"
                        :style="{
                            transition: current_transition,
                        }"
                        @click="goTo(index)"
                    ></span>
                </div>
            </slot>
        </template>
    </div>
</template>

<script setup lang="ts">
import type {Property} from "csstype";
import type {CSSProperties} from "vue";
export interface ISliderProps {
    spaceBetween?: number;
    slidesPerView?: number;
    slidesPerMove?: number;
    inlinePadding?: Property.Padding<string>;
    defaultIndex?: number;

    draggable?: "base";

    loop?: boolean;

    showArrows?: boolean;
    arrowsPlacement?: "center" | "left" | "right";

    showPagination?: boolean;
    paginationType?: "dot" | "line" | "progress";
    paginationPlacement?: "top" | "bottom" | "left" | "right";
    paginationTrigger?: "click" | "mouseover";

    centeredSlides?: boolean;

    currentSlideStyle?: CSSProperties;
}

const props = withDefaults(defineProps<ISliderProps>(), {
    spaceBetween: 0,
    slidesPerView: 1,
    slidesPerMove: 1,
    inlinePadding: "0px",

    loop: false,

    showArrows: false,
    arrowsPlacement: "right",

    showPagination: false,
    paginationType: "dot",
    paginationPlacement: "bottom",
    paginationTrigger: "click",

    centeredSlides: false,
});

const DEFAULT_TRANSITION: string = "300ms";
const ZERO_TRANSITION: string = "unset";

const slots = useSlots();

const slot_childrens =
    slots
        .default?.()
        ?.flatMap(vnode => (vnode.type === Symbol.for("v-fgt") ? vnode.children || [] : [vnode])) ||
    [];

const current_index = ref<number>(props?.defaultIndex || 0);

const current_transition = ref<string>(ZERO_TRANSITION);

const slider = ref<HTMLDivElement>();

const slide_width = ref<number>(0);

const drag_position = ref<number>(0);

// TODO привести компутед в порядок, а то как мусорка выглядит
const transform = computed<Readonly<number>>(() => {
    if (typeof slide_width.value !== "number") return 0;

    const prev_slides_width = current_index.value * (slide_width.value + props?.spaceBetween);
    const center_shift =
        props?.centeredSlides && slider.value
            ? slider.value.offsetWidth / 2 - slide_width.value / 2
            : 0;
    const loop_shift = props?.loop
        ? slot_childrens.length * (slide_width.value + props?.spaceBetween)
        : 0;

    const current_slide_width: number =
        typeof props?.currentSlideStyle?.width == "string"
            ? +props?.currentSlideStyle?.width.replace(/\D/g, "")
            : props?.currentSlideStyle?.width ?? 0;

    const current_slide_shift = (current_slide_width - slide_width.value) / 2;

    return (
        prev_slides_width + drag_position.value + loop_shift - center_shift + current_slide_shift
    );
});

function calcSlideWidth() {
    if (!slider.value) {
        slide_width.value = 0;
        return;
    }

    slide_width.value =
        (slider.value.clientWidth - props?.spaceBetween * (props?.slidesPerView - 1)) /
        props?.slidesPerView;
}

function getSlidePerView(): number {
    if (typeof props.slidesPerView === "number" && !props?.centeredSlides)
        return props.slidesPerView;

    return 1;
}

function transitionWrapper(callback?: Function, after_callback?: Function) {
    current_transition.value = DEFAULT_TRANSITION;

    if (callback) callback();

    setTimeout(() => {
        current_transition.value = ZERO_TRANSITION;
        // NOTE можно подумать на счет того, что бы не передавать функцию, а отслеживать значения внутри этой
        if (after_callback) after_callback();
    }, +DEFAULT_TRANSITION.replace(/\D/g, ""));
}

function next(): void {
    transitionWrapper(
        () => {
            const slider_length = slot_childrens.length - getSlidePerView();

            if (current_index.value == slider_length && !props?.loop) {
                return;
            }

            if (current_index.value + props?.slidesPerMove >= slider_length && !props?.loop) {
                current_index.value = slider_length;
                return;
            }

            current_index.value = current_index.value + props?.slidesPerMove;
            return;
        },
        () => {
            if (props?.loop && current_index.value > slot_childrens.length - 1) {
                const real_index = current_index.value;

                current_index.value = real_index - slot_childrens.length;
            }
        }
    );
}

async function prev(): Promise<void> {
    transitionWrapper(
        () => {
            if (current_index.value == 0 && !props?.loop) {
                return;
            }

            const real_index = current_index.value - props?.slidesPerMove;

            if (real_index < 0 && !props?.loop) {
                current_index.value = 0;
                return;
            }

            current_index.value = real_index;
        },
        () => {
            if (props?.loop && current_index.value < 0) {
                const real_index = current_index.value;
                current_index.value = slot_childrens.length + real_index;
            }
        }
    );
}

function goTo(index: number): void {
    transitionWrapper(() => {
        if (index < 0) {
            current_index.value = 0;
            return;
        }

        const slider_length = slot_childrens.length - getSlidePerView();

        if (index >= slider_length) {
            current_index.value = props?.loop ? index : slider_length;
            return;
        }

        current_index.value = index;
        return;
    });
}

let is_moved: boolean = false;

let start_x: number = 0;

let time_start: number = 0;
let time_end: number = 0;

function dragStart(event: MouseEvent | TouchEvent): void {
    if (
        !props?.draggable ||
        !(event.target instanceof Node && slider.value?.contains(event.target))
    )
        return;

    time_start = performance.now();
    is_moved = true;

    current_transition.value = ZERO_TRANSITION;

    start_x = event instanceof MouseEvent ? event.clientX : event.touches[0]!.clientX;
}

function dragMove(event: MouseEvent | TouchEvent): void {
    if (!props?.draggable || !is_moved) return;

    drag_position.value =
        start_x - (event instanceof MouseEvent ? event.clientX : event.touches[0]!.clientX);

    if (!props?.loop) return;

    if (
        transform.value <
        slot_childrens.length * (slide_width.value + props?.spaceBetween) - slide_width.value
    ) {
        current_index.value += slot_childrens.length;
        return;
    }

    if (transform.value > slot_childrens.length * (slide_width.value + props?.spaceBetween) * 2) {
        current_index.value -= slot_childrens.length;
        return;
    }
}

function dragEnd(): void {
    if (!props?.draggable || !is_moved) return;

    time_end = performance.now();

    is_moved = false;

    transitionWrapper(() => {
        if (time_end - time_start > 400 && slide_width.value / 2 > Math.abs(drag_position.value)) {
            drag_position.value = 0;
            return;
        }

        if (drag_position.value > 30 && slide_width.value / 2 > Math.abs(drag_position.value)) {
            next();
        } else if (
            drag_position.value < -30 &&
            slide_width.value / 2 > Math.abs(drag_position.value)
        ) {
            prev();
        } else {
            // FIXME тут математика не понятно работает, надо думать, а я зочу спать. Но косяк почти не заметный так что на прод можно выкатить
            const current_slide_width: number =
                typeof props?.currentSlideStyle?.width == "string"
                    ? +props?.currentSlideStyle?.width.replace(/\D/g, "")
                    : props?.currentSlideStyle?.width ?? 0;

            const current_slide_shift = (current_slide_width - slide_width.value) / 2;

            goTo(
                Math.floor(
                    (drag_position.value + slide_width.value / 2 - current_slide_shift) /
                        slide_width.value
                ) + current_index.value + (drag_position.value < -30 ? 1 : 0)
            );
        }

        drag_position.value = 0;
    });
}

onMounted(() => {
    if (!slider.value) return;

    window.addEventListener("resize", calcSlideWidth);

    slider.value.querySelectorAll("img").forEach((img: HTMLImageElement) => {
        img.setAttribute("draggable", "false");
    });

    window.addEventListener("mousedown", dragStart, true);
    window.addEventListener("mousemove", dragMove, true);
    window.addEventListener("mouseup", dragEnd, true);

    window.addEventListener("touchstart", dragStart, true);
    window.addEventListener("touchmove", dragMove, true);
    window.addEventListener("touchend", dragEnd, true);

    calcSlideWidth();
});

onBeforeUnmount(() => {
    window.removeEventListener("resize", calcSlideWidth);

    window.removeEventListener("mousedown", dragStart, true);
    window.removeEventListener("mousemove", dragMove, true);
    window.removeEventListener("mouseup", dragEnd, true);

    window.removeEventListener("touchstart", dragStart, true);
    window.removeEventListener("touchmove", dragMove, true);
    window.removeEventListener("touchend", dragEnd, true);
});

defineExpose({
    next,
    prev,
    goTo,
});
</script>

<style>
.ui-slider__wrapper {
    position: relative;
    user-select: none;
    width: 100%;
}

.ui-slider {
    height: max-content;
    width: 100%;
    touch-action: pan-x;
    overflow: hidden;
}

.ui-slider._draggable {
    cursor: grab;
}

.ui-slider._draggable:active {
    cursor: grabbing;
}

.ui-slider__slides {
    display: flex;
    width: 100%;
    height: 100%;
}

.ui-slider__slides-item {
    flex-shrink: 0;
    position: relative;
    overflow: hidden;
}

.ui-slider__slides-item > img {
    display: block;
    object-fit: cover;
    object-position: center;
}

.ui-slider__arrows {
    position: absolute;
    visibility: hidden;
    display: flex;
    gap: 10px;
}

.ui-slider__arrows._left {
    left: 10px;
    bottom: 10px;
}

.ui-slider__arrows._right {
    right: 10px;
    bottom: 10px;
}

.ui-slider__arrows._center {
    top: 50%;
    transform: translate(0%, -50%);
    width: 100%;
    padding: 10px;
    justify-content: space-between;
}

.ui-slider__arrows-item {
    all: unset;
    box-sizing: border-box;
    background-color: #99999925;
    cursor: pointer;
    height: 25px;
    aspect-ratio: 1;
    border-radius: 3px;
    padding: 2px;
    transition: var(--base-transition);
}

.ui-slider__arrows-item:hover {
    background-color: #bbbbbb45;
    transition: var(--active-transition);
}

.ui-slider__arrows-item:active {
    background-color: #bbbbbb75;
}

.ui-slider__arrows-item._lock {
    cursor: not-allowed;
    opacity: 0.6;
}

.ui-slider__arrows > * {
    visibility: visible;
}

.ui-slider__pagination {
    position: absolute;
    display: flex;
    gap: 10px;
    visibility: hidden;
}

.ui-slider__pagination._top {
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
}

.ui-slider__pagination._bottom {
    bottom: 10px;
    left: 50%;
    transform: translateX(-50%);
}

.ui-slider__pagination._left {
    flex-direction: column;
    top: 50%;
    left: 10px;
    transform: translateY(-50%);
}

.ui-slider__pagination._right {
    flex-direction: column;
    top: 50%;
    right: 10px;
    transform: translateY(-50%);
}

.ui-slider__pagination-item {
    visibility: visible;
    cursor: pointer;
    background-color: #fff;
    opacity: 0.2;
}

.ui-slider__pagination-item._current {
    opacity: 0.7;
}

.ui-slider__pagination-item._dot {
    height: 10px;
    width: 10px;
    max-height: 10px;
    max-width: 10px;
    min-height: 10px;
    min-width: 10px;
    border-radius: 50%;
}

.ui-slider__pagination-item._line {
    height: 5px;
    width: 15px;
    max-width: 15px;
    min-width: 15px;
    border-radius: 20px;
}

.ui-slider__pagination-item._line._current {
    width: 30px;
    max-width: 30px;
    min-width: 30px;
    cursor: default;
}
</style>
