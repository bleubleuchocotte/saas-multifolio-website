<script setup lang="ts">
type ComponentProps = {
	/**
	 * The key used for managing the state in the component. It must be **unique** through the whole app.
	 */
	stateKey: string
};

const props = defineProps<ComponentProps>();

const isOpen = ref(false);
defineExpose({ isOpen, updateHeight });

const id = useState(props.stateKey, () => {
	return {
		header: `accordion-header-${Math.random().toString(36).substring(2, 9)}`,
		content: `accordion-content-${Math.random().toString(36).substring(2, 9)}`,
	};
});

const toggleAccordion = useThrottleFn(() => {
	isOpen.value = !isOpen.value;
}, 100);

const isButtonFocused = ref(false);
const target = ref<HTMLDivElement | null>(null);
const height = ref(0);

onMounted(() => {
	updateHeight();
});

function updateHeight() {
	if (target.value) {
		height.value = target.value.scrollHeight;
	}
}
</script>

<template>
	<div class="accordion" :style="`--accordion-height: ${height}px`">
		<UIButton
			:id="id.header"
			type="button"
			:aria-expanded="isOpen ? 'true' : 'false'"
			:aria-controls="id.content"
			class="accordion__button"
			@click="toggleAccordion"
			@focus="isButtonFocused = true"
			@blur="isButtonFocused = false"
		>
			<slot name="title" :focus="isButtonFocused || isOpen" />
		</UIButton>
		<div
			:id="id.content"
			ref="target"
			role="region"
			:aria-labelledby="id.header"
			:aria-hidden="isOpen ? 'false' : 'true'"
			class="accordion__content"
		>
			<slot name="content" />
		</div>
	</div>
	<details class="accordion-no-script">
		<summary>
			<slot name="title" />
		</summary>
		<slot name="content" />
	</details>
</template>

<style scoped lang="scss">
.accordion {
	&-no-script {
		display: none;
	}

	@media (scripting: none) {
		display: none;

		&-no-script {
			display: initial;
		}
	}

	&__content {
		max-height: 0;
		overflow: hidden;

		@include transition();

		&[aria-hidden="false"] {
			max-height: var(--accordion-height);
		}
	}
}
</style>
