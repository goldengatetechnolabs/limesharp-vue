<template>
	<div class="grid md:grid-cols-2 grid-cols-1 md:h-screen">

		<!-- LEFT: static image by default, project image when a project is open -->
		<div class="md:h-screen flex items-center justify-center overflow-hidden">
			<img :src="activeImage" ref="mainImage" class="main-img" alt="Limesharp" title="Limesharp" />
		</div>

		<!-- RIGHT -->
		<div class="md:h-screen overflow-y-auto right">

			<!-- Screen 1: logo + project list (hidden during transition AND when project is open) -->
			<div class="p-6" v-if="!activeProject && !isTransitioning">
				<div ref="logoRef">
					<img :src="logo" class="logo-img mx-auto mb-20 lg:pb-20 w-full" alt="Limesharp" title="Limesharp" />
				</div>

				<div class="projects" ref="projectItems">
					<div class="section-title">
						<h3 class="text-2xl font-medium text-center mb-10">
							Redefining the future of <span class="block">Digital Commerce</span>
						</h3>
					</div>

					<div class="grid lg:grid-cols-2 grid-cols-1 gap-6">
						<div v-for="(project, i) in projects" :key="i" class="project-box cursor-pointer"
							@click="openProject(project, $event)">
							<div class="project-img mb-2">
								<img :src="project.image" class="" :alt="project.title" :title="project.title" />
							</div>
							<div class="flex justify-between">
								<h3 class="font-medium">{{ project.title }}</h3>
								<div class="text-base font-heading">{{ project.date }}</div>
							</div>
						</div>
					</div>
				</div>
			</div>

			<!-- Screen 2: starts invisible (opacity:0), animated in AFTER image lands -->
			<div v-else-if="activeProject && !isTransitioning" ref="detailPanel"
				class="project-details lg:p-20 lg:pb-6 p-6 text-center md:h-screen flex flex-col items-center justify-between"
				style="opacity: 0;">
				<div ref="detailTitle">
					<h2 class="text-5xl lg:text-6xl font-medium md:mb-4 mb-2">{{ activeProject.title }}</h2>
					<div class="section-title text-2xl"><span>{{ activeProject.date }}</span></div>
				</div>
				<p ref="detailDesc" class="text-xl lg:text-2xl md:my-6 my-20 lg:px-10">{{ activeProject.desc }}</p>
				<button ref="detailBtn" @click="closeProject" class="btn-back">Go Back</button>
			</div>

		</div>
	</div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

import logo from './images/limesharp-logo.svg'
import mainImg from './images/main.jpg'
import project01 from './images/project01.jpg'
import project02 from './images/project02.jpg'
import project03 from './images/project03.jpg'
import project04 from './images/project04.jpg'
import project05 from './images/project05.jpg'
import project06 from './images/project06.jpg'

gsap.registerPlugin(ScrollTrigger)

const projects = ref([
	{ title: 'Project 01', date: '01.07', image: project01, desc: 'Lorem Ipsum is simply dummy text of the printing and typesetting industry' },
	{ title: 'Project 02', date: '02.07', image: project02, desc: "Lorem Ipsum has been the industry's standard dummy text ever since the 1500s" },
	{ title: 'Project 03', date: '03.07', image: project03, desc: 'It is a long established fact that a reader will be distracted by the readable content of a page when looking at its layout.' },
	{ title: 'Project 04', date: '04.07', image: project04, desc: 'There are many variations of passages of Lorem Ipsum available, but the majority have suffered alteration in some form' },
	{ title: 'Project 05', date: '05.07', image: project05, desc: 'The standard chunk of Lorem Ipsum used since the 1500s is reproduced below for those interested' },
	{ title: 'Project 06', date: '06.07', image: project06, desc: "If you are going to use a passage of Lorem Ipsum, you need to be sure there isn't anything embarrassing hidden in the middle of text." },
])

const activeImage = ref(mainImg)
const activeProject = ref(null)
const isTransitioning = ref(false)

// Template refs
const mainImage = ref(null)
const logoRef = ref(null)
const projectItems = ref(null)
const detailPanel = ref(null)
const detailTitle = ref(null)
const detailDesc = ref(null)
const detailBtn = ref(null)

let lastThumbRect = null

// Screen 1: left image + logo + project list fade-in-up
const animateScreen1 = () => {
	const els = [mainImage.value, logoRef.value, projectItems.value]
	gsap.killTweensOf(els)
	gsap.set(els, { clearProps: 'all' })

	gsap.from(els, {
		y: 40,
		opacity: 0,
		duration: 0.8,
		ease: 'power3.out',
		stagger: 0.2,
	})
}

// Screen 2: title → desc → button staggered fade-in-up 
const animateScreen2 = () => {
	// Reveal the panel (was opacity:0 to hide before animation)
	gsap.set(detailPanel.value, { opacity: 1 })

	gsap.fromTo(
		[detailTitle.value, detailDesc.value, detailBtn.value],
		{ y: 50, opacity: 0 },
		{ y: 0, opacity: 1, duration: 0.7, ease: 'power3.out', stagger: 0.15 }
	)
}

//  Open project 
const openProject = async (project, event) => {
	const img = event.currentTarget.querySelector('img')
	const rect = img.getBoundingClientRect()

	lastThumbRect = { ...rect }

	// Clone thumbnail and pin it over its original position BEFORE anything fades
	const clone = img.cloneNode()
	document.body.appendChild(clone)

	Object.assign(clone.style, {
		position: 'fixed',
		top: rect.top + 'px',
		left: rect.left + 'px',
		width: rect.width + 'px',
		height: rect.height + 'px',
		zIndex: 999,
		margin: 0,
		objectFit: 'cover',
		transition: 'none',
	})

	// Step 1 — fade out left static image + logo + project list together
	await gsap.to([mainImage.value, logoRef.value, projectItems.value], {
		opacity: 0,
		y: -0,
		duration: 0,
		ease: 'power2.in',
		stagger: 0,
	})

	// Step 2 — remove Screen 1 from DOM while clone is flying
	isTransitioning.value = true
	await nextTick()

	const target = mainImage.value.getBoundingClientRect()

	// Step 3 — image flies to the left panel
	gsap.to(clone, {
		top: target.top,
		left: target.left,
		width: target.width,
		height: target.height,
		duration: 0.8,
		ease: 'power3.inOut',
		onComplete: async () => {
			// Step 4 — swap image source and set real project (renders Screen 2, still opacity:0)
			activeImage.value = project.image
			activeProject.value = project
			isTransitioning.value = false
			clone.remove()

			// Restore left image visibility with the new project image
			await nextTick()
			gsap.set(mainImage.value, { opacity: 1, y: 0 })

			// Step 5 — wait for Vue to render Screen 2, then animate content in
			await nextTick()
			animateScreen2()
		},
	})
}

//  Go Back 
const closeProject = async () => {
	const rect = mainImage.value.getBoundingClientRect()

	// Clone the project image — it will fly back while both screens are hidden
	const clone = mainImage.value.cloneNode()
	document.body.appendChild(clone)

	Object.assign(clone.style, {
		position: 'fixed',
		top: rect.top + 'px',
		left: rect.left + 'px',
		width: rect.width + 'px',
		height: rect.height + 'px',
		zIndex: 999,
		margin: 0,
		objectFit: 'cover',
		transition: 'none',
	})

	// Hide left image — clone takes over visually
	gsap.set(mainImage.value, { opacity: 0 })

	// isTransitioning = true hides BOTH screens while clone is flying
	isTransitioning.value = true
	activeProject.value = null
	activeImage.value = mainImg

	await nextTick()

	// Fly clone back to thumbnail position
	const flyBack = lastThumbRect
		? new Promise(resolve => {
			gsap.to(clone, {
				top: lastThumbRect.top,
				left: lastThumbRect.left,
				width: lastThumbRect.width,
				height: lastThumbRect.height,
				duration: 0.2,
				ease: 'power3.inOut',
				onComplete: resolve,
			})
		})
		: Promise.resolve()

	await flyBack
	clone.remove()
	lastThumbRect = null

	// Now reveal Screen 1 and animate everything in
	isTransitioning.value = false
	await nextTick()
	animateScreen1()
}

//  Initial mount 
onMounted(() => {
	animateScreen1()
})
</script>