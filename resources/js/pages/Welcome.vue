<script setup lang="ts">
import { Head, Link } from '@inertiajs/vue3';
import { login, register } from '@/routes';
import { MapPin, Heart, BookOpen, Users, ArrowRight, Facebook, Play, Baby, Sparkles, Shield, Zap, Flame } from 'lucide-vue-next';
import { Button } from '@/components/ui/button';
import { Separator } from '@/components/ui/separator';
import SiteHeader from '@/components/SiteHeader.vue';
import { ref, reactive, onMounted, onUnmounted, computed, nextTick } from 'vue';

withDefaults(
    defineProps<{
        canRegister: boolean;
    }>(),
    {
        canRegister: true,
    },
);

// ── State ──────────────────────────────────────────────
const heroSection = ref<HTMLElement | null>(null);
const heroCanvas = ref<HTMLCanvasElement | null>(null);
const heroReady = ref(false);
const prefersReducedMotion = ref(false);

const mouse = reactive({ x: 0.5, y: 0.5, targetX: 0.5, targetY: 0.5 });

let animationFrameId: number | null = null;
let ctx: CanvasRenderingContext2D | null = null;
let canvasW = 0;
let canvasH = 0;

// ── Gentle golden motes ────────────────────────────────
interface Mote {
    x: number;
    y: number;
    vy: number;
    size: number;
    opacity: number;
    maxOpacity: number;
    life: number;
    maxLife: number;
    drift: number;
    driftSpeed: number;
}

const motes: Mote[] = [];
const MOTE_COUNT = 20;

function spawnMote(randomY = false): Mote {
    return {
        x: Math.random() * canvasW,
        y: randomY ? Math.random() * canvasH : canvasH + 10,
        vy: -(0.15 + Math.random() * 0.35),
        size: 0.8 + Math.random() * 1.5,
        opacity: 0,
        maxOpacity: 0.15 + Math.random() * 0.25,
        life: randomY ? Math.random() * 600 : 0,
        maxLife: 500 + Math.random() * 400,
        drift: Math.random() * Math.PI * 2,
        driftSpeed: 0.003 + Math.random() * 0.005,
    };
}

function initMotes(): void {
    motes.length = 0;
    for (let i = 0; i < MOTE_COUNT; i++) {
        motes.push(spawnMote(true));
    }
}

// ── Canvas loop ────────────────────────────────────────
function renderCanvas(): void {
    if (!ctx) return;

    ctx.clearRect(0, 0, canvasW, canvasH);

    // Smooth mouse interpolation
    mouse.x += (mouse.targetX - mouse.x) * 0.03;
    mouse.y += (mouse.targetY - mouse.y) * 0.03;

    ctx.globalCompositeOperation = 'lighter';

    for (let i = 0; i < motes.length; i++) {
        const m = motes[i];
        m.life++;

        // Lifecycle fade
        const ratio = m.life / m.maxLife;
        if (ratio < 0.2) {
            m.opacity = (ratio / 0.2) * m.maxOpacity;
        } else if (ratio > 0.75) {
            m.opacity = ((1 - ratio) / 0.25) * m.maxOpacity;
        }

        // Drift
        m.drift += m.driftSpeed;
        m.x += Math.sin(m.drift) * 0.2;
        m.y += m.vy;

        // Soft glow
        const glow = ctx.createRadialGradient(m.x, m.y, 0, m.x, m.y, m.size * 3);
        glow.addColorStop(0, `hsla(40, 90%, 80%, ${m.opacity})`);
        glow.addColorStop(0.5, `hsla(38, 85%, 70%, ${m.opacity * 0.3})`);
        glow.addColorStop(1, 'hsla(36, 80%, 60%, 0)');

        ctx.beginPath();
        ctx.arc(m.x, m.y, m.size * 3, 0, Math.PI * 2);
        ctx.fillStyle = glow;
        ctx.fill();

        // Recycle
        if (m.life >= m.maxLife || m.y < -10) {
            motes[i] = spawnMote(false);
        }
    }

    ctx.globalCompositeOperation = 'source-over';
    animationFrameId = requestAnimationFrame(renderCanvas);
}

function resizeCanvas(): void {
    if (!heroCanvas.value || !heroSection.value) return;

    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    const rect = heroSection.value.getBoundingClientRect();
    heroCanvas.value.width = rect.width * dpr;
    heroCanvas.value.height = rect.height * dpr;
    heroCanvas.value.style.width = `${rect.width}px`;
    heroCanvas.value.style.height = `${rect.height}px`;

    if (ctx) {
        ctx.scale(dpr, dpr);
    }
    canvasW = rect.width;
    canvasH = rect.height;
}

// ── Events ─────────────────────────────────────────────
function handleMouseMove(e: MouseEvent): void {
    if (!heroSection.value || window.innerWidth < 768) return;
    const rect = heroSection.value.getBoundingClientRect();
    mouse.targetX = (e.clientX - rect.left) / rect.width;
    mouse.targetY = (e.clientY - rect.top) / rect.height;
}

// ── Lifecycle ──────────────────────────────────────────
onMounted(async () => {
    if (typeof window === 'undefined') return;

    prefersReducedMotion.value = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

    await nextTick();
    setTimeout(() => { heroReady.value = true; }, 100);

    if (prefersReducedMotion.value) return;

    if (heroCanvas.value) {
        ctx = heroCanvas.value.getContext('2d');
        resizeCanvas();
        initMotes();
        renderCanvas();
    }

    window.addEventListener('mousemove', handleMouseMove, { passive: true });
    window.addEventListener('resize', resizeCanvas, { passive: true });
});

onUnmounted(() => {
    if (typeof window === 'undefined') return;
    if (animationFrameId) cancelAnimationFrame(animationFrameId);
    window.removeEventListener('mousemove', handleMouseMove);
    window.removeEventListener('resize', resizeCanvas);
});

// ── Computed ───────────────────────────────────────────
const parallaxStyle = computed(() => {
    if (prefersReducedMotion.value) return {};
    const px = (mouse.x - 0.5) * 12;
    const py = (mouse.y - 0.5) * 12;
    return { transform: `translate(${px}px, ${py}px) scale(1.06)` };
});
</script>

<template>
    <Head title="New Beginnings Assembly of God — Grayson, KY">
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
        <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400;1,500&family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet" />
    </Head>

    <style scoped>
    /* ── Staggered fade-in ─────────────────────────────── */
    @keyframes hero-fade-up {
        from {
            opacity: 0;
            transform: translateY(16px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }

    .hero-enter-0,
    .hero-enter-1,
    .hero-enter-2,
    .hero-enter-3,
    .hero-enter-4 {
        opacity: 0;
    }

    .hero-ready .hero-enter-0 { animation: hero-fade-up 700ms cubic-bezier(0.22, 1, 0.36, 1) 150ms both; }
    .hero-ready .hero-enter-1 { animation: hero-fade-up 700ms cubic-bezier(0.22, 1, 0.36, 1) 300ms both; }
    .hero-ready .hero-enter-2 { animation: hero-fade-up 700ms cubic-bezier(0.22, 1, 0.36, 1) 450ms both; }
    .hero-ready .hero-enter-3 { animation: hero-fade-up 700ms cubic-bezier(0.22, 1, 0.36, 1) 600ms both; }
    .hero-ready .hero-enter-4 { animation: hero-fade-up 700ms cubic-bezier(0.22, 1, 0.36, 1) 750ms both; }

    @media (prefers-reduced-motion: reduce) {
        .hero-enter-0, .hero-enter-1, .hero-enter-2,
        .hero-enter-3, .hero-enter-4 {
            animation: none !important;
            opacity: 1 !important;
            transform: none !important;
        }
    }
    </style>

    <div class="min-h-screen bg-background text-foreground antialiased">

        <!-- ============================================ -->
        <!-- ABOVE THE FOLD: Header + Hero               -->
        <!-- ============================================ -->

        <SiteHeader />

        <!-- Hero -->
        <section
            ref="heroSection"
            :class="['relative flex min-h-[88dvh] items-center overflow-hidden md:min-h-[92dvh]', heroReady ? 'hero-ready' : '']"
        >
            <!-- Background video -->
            <video
                autoplay
                loop
                muted
                playsinline
                poster="https://images.unsplash.com/photo-1477238134895-98438ad85c30?auto=format&fit=crop&w=1920&q=80"
                class="absolute inset-0 h-full w-full object-cover will-change-transform"
                :style="parallaxStyle"
            >
                <source src="/hero_shot.mp4" type="video/mp4" />
            </video>

            <!-- Gradient overlay for readability -->
            <div class="absolute inset-0 bg-gradient-to-b from-black/60 via-black/40 to-black/70" />

            <!-- Warm color wash -->
            <div class="absolute inset-0 bg-gradient-to-br from-amber-900/20 via-transparent to-transparent mix-blend-screen" />

            <!-- Canvas — gentle floating motes -->
            <canvas
                ref="heroCanvas"
                class="pointer-events-none absolute inset-0 z-[1] mix-blend-screen"
            />

            <!-- Content -->
            <div class="relative z-10 mx-auto w-full max-w-2xl px-6 pt-20 pb-24 text-left md:pt-28">
                <!-- Headline - stacked lines -->
                <div class="space-y-4" aria-label="Welcome message emphasizing everyone is welcome">
                    <h1 class="hero-enter-1 text-5xl font-bold leading-[1.1] tracking-tight text-white sm:text-6xl" style="font-family: 'Inter', sans-serif;">
                        No perfect people.
                    </h1>
                    <h2 class="hero-enter-2 text-4xl font-bold leading-[1.1] tracking-tight text-white sm:text-5xl" style="font-family: 'Inter', sans-serif;">
                        No prerequisites.
                    </h2>
                    <p class="hero-enter-3 text-lg font-medium leading-relaxed text-white/80 sm:text-xl" style="font-family: 'Inter', sans-serif;">
                        Just grace — and a place to belong.
                    </p>
                </div>

                <!-- Service times meta pill -->
                <div class="hero-enter-3 mt-6 max-w-2xl">
                    <div
                        class="inline-flex items-center gap-2 rounded-full border border-white/15 bg-white/10 px-4 py-2 text-sm font-semibold text-white shadow-sm backdrop-blur"
                        aria-label="Service times and location: Sundays 10:30 AM, Wednesdays 7 PM, Grayson Kentucky"
                    >
                        Sundays 10:30 AM • Wednesdays 7 PM • Grayson, KY
                    </div>
                </div>

                <!-- CTA Row -->
                <div class="hero-enter-4 mt-8 flex flex-col items-start gap-3 sm:flex-row sm:gap-4">
                    <!-- Primary CTA -->
                    <a href="#visit">
                        <Button
                            size="lg"
                            class="h-13 rounded-full bg-white px-10 text-[15px] font-semibold text-gray-900 shadow-xl shadow-black/25 transition-all duration-300 hover:-translate-y-0.5 hover:shadow-2xl active:translate-y-0"
                        >
                            Plan Your Visit
                            <ArrowRight class="ml-2 h-4.5 w-4.5" />
                        </Button>
                    </a>

                    <!-- Secondary CTA -->
                    <a href="/live" aria-label="Watch live service stream">
                        <button
                            class="inline-flex h-13 items-center justify-center rounded-xl border border-white/25 bg-white/0 px-10 text-[15px] font-semibold text-white backdrop-blur transition-all hover:bg-white/10"
                        >
                            <Play class="mr-2 h-4.5 w-4.5 opacity-90" />
                            Watch Live
                        </button>
                    </a>
                </div>
            </div>

            <!-- Bottom fade -->
            <div class="pointer-events-none absolute inset-x-0 bottom-0 z-10 h-32 bg-gradient-to-t from-background to-transparent" />
        </section>

        <!-- Spacer for breathing room -->
        <div class="h-16 sm:h-24" />

        <!-- ============================================ -->
        <!-- BELOW THE FOLD                               -->
        <!-- ============================================ -->

        <!-- Join Us This Week -->
        <section id="services" class="relative py-20 sm:py-28">
            <div class="mx-auto max-w-6xl px-6">
                <!-- Section heading -->
                <div class="mx-auto max-w-2xl text-center">
                    <h2 class="text-3xl font-bold tracking-tight sm:text-4xl" style="font-family: 'Playfair Display', serif;">
                        Join Us This Week
                    </h2>
                    <p class="mx-auto mt-5 max-w-xl text-base leading-relaxed text-muted-foreground">
                        We gather each week to worship, grow, and encourage one another in Christ. 
                        Whether you're new to church or looking for a place to belong, you're welcome here.
                    </p>
                </div>

                <!-- Service times cards -->
                <div class="mx-auto mt-14 grid max-w-4xl gap-6 sm:grid-cols-2">
                    <div class="rounded-2xl border border-primary/10 bg-card p-8 text-center transition-all hover:border-primary/20 hover:shadow-md hover:shadow-primary/5">
                        <p class="text-sm font-semibold text-muted-foreground">Sunday Morning</p>
                        <p class="mt-3 text-4xl font-bold tracking-tight text-foreground" style="font-family: 'Playfair Display', serif;">10:30 <span class="text-xl">AM</span></p>
                        <p class="mt-2 text-sm text-muted-foreground">Worship Service</p>
                    </div>

                    <div class="rounded-2xl border border-primary/10 bg-card p-8 text-center transition-all hover:border-primary/20 hover:shadow-md hover:shadow-primary/5">
                        <p class="text-sm font-semibold text-muted-foreground">Wednesday</p>
                        <p class="mt-3 text-4xl font-bold tracking-tight text-foreground" style="font-family: 'Playfair Display', serif;">7:00 <span class="text-xl">PM</span></p>
                        <p class="mt-2 text-sm text-muted-foreground">Bible Study</p>
                    </div>
                </div>

                <!-- Scripture -->
                <div class="mx-auto mt-16 max-w-2xl text-center">
                    <blockquote>
                        <p class="text-lg leading-relaxed italic text-muted-foreground/70" style="font-family: 'Playfair Display', serif;">
                            "Therefore, if anyone is in Christ, the new creation has come: 
                            The old has gone, the new is here!"
                        </p>
                        <footer class="mt-3">
                            <p class="text-xs font-medium uppercase tracking-[0.15em] text-muted-foreground/60">2 Corinthians 5:17</p>
                        </footer>
                    </blockquote>
                </div>
            </div>
        </section>

        <!-- Our Heart for People — Image Background -->
        <section id="about" class="relative isolate overflow-hidden">
            <!-- Background image -->
            <img
                src="https://images.unsplash.com/photo-1511632765486-a01980e01a18?auto=format&fit=crop&w=1920&q=80"
                alt="People gathering and connecting in community"
                class="absolute inset-0 h-full w-full object-cover"
            />

            <!-- Dark gradient overlay for readability -->
            <div class="absolute inset-0 bg-gradient-to-b from-black/55 via-black/45 to-black/55" />

            <!-- Content layer -->
            <div class="relative mx-auto max-w-5xl px-6 py-16 sm:py-24 text-center">
                <div class="mx-auto max-w-2xl">
                    <h2 class="text-lg font-semibold tracking-wide text-white/90 sm:text-xl" style="font-family: 'Playfair Display', serif;">
                        Rooted in Faith.<br />Focused on People.
                    </h2>
                    <p class="mt-5 text-base leading-relaxed text-white/85 sm:text-lg">
                        We believe church should be a place where people feel known, valued, and encouraged — no matter where they are on their faith journey. At New Beginnings Assembly of God, our heart is to love people well, teach God's Word faithfully, and live out our faith in practical ways every day.
                    </p>
                </div>

                <!-- Value cards with frosted glass effect -->
                <div class="mx-auto mt-10 grid max-w-5xl gap-4 sm:gap-6 md:grid-cols-3">
                    <div class="rounded-2xl border border-white/15 bg-white/10 p-6 shadow-lg shadow-black/10 backdrop-blur-sm transition-all hover:bg-white/15">
                        <div class="mx-auto flex h-14 w-14 items-center justify-center rounded-xl bg-white/15">
                            <BookOpen class="h-6 w-6 text-white" />
                        </div>
                        <h3 class="mt-5 text-base font-semibold text-white">Biblical Teaching</h3>
                        <p class="mt-2 text-sm font-medium text-white/85">Clear, practical teaching rooted in Scripture</p>
                        <p class="mt-3 text-sm leading-relaxed text-white/80">
                            We teach the Bible in a way that's faithful to God's Word and relevant to everyday life — helping you grow in faith, understanding, and purpose.
                        </p>
                    </div>

                    <div class="rounded-2xl border border-white/15 bg-white/10 p-6 shadow-lg shadow-black/10 backdrop-blur-sm transition-all hover:bg-white/15">
                        <div class="mx-auto flex h-14 w-14 items-center justify-center rounded-xl bg-white/15">
                            <Users class="h-6 w-6 text-white" />
                        </div>
                        <h3 class="mt-5 text-base font-semibold text-white">Genuine Community</h3>
                        <p class="mt-2 text-sm font-medium text-white/85">A place to belong, not just attend</p>
                        <p class="mt-3 text-sm leading-relaxed text-white/80">
                            Church is more than a service. It's a family. We believe in building real relationships where people are supported, encouraged, and never walk alone.
                        </p>
                    </div>

                    <div class="rounded-2xl border border-white/15 bg-white/10 p-6 shadow-lg shadow-black/10 backdrop-blur-sm transition-all hover:bg-white/15">
                        <div class="mx-auto flex h-14 w-14 items-center justify-center rounded-xl bg-white/15">
                            <Heart class="h-6 w-6 text-white" />
                        </div>
                        <h3 class="mt-5 text-base font-semibold text-white">Compassionate Outreach</h3>
                        <p class="mt-2 text-sm font-medium text-white/85">Living our faith beyond the church walls</p>
                        <p class="mt-3 text-sm leading-relaxed text-white/80">
                            Our faith doesn't stop at Sunday. We serve Carter County and beyond by meeting needs, showing compassion, and sharing the hope of Christ through action.
                        </p>
                    </div>
                </div>

                <!-- Bridge line -->
                <div class="mx-auto mt-10 max-w-2xl">
                    <p class="text-base leading-relaxed italic text-white/85">
                        If you're looking for a church where faith is lived out with love and authenticity, we'd love for you to be part of our church family.
                    </p>
                </div>
            </div>
        </section>

        <!-- Plan Your Visit — Premium Feature Panel -->
        <section class="bg-[#f5f5f4] py-20">
            <div class="mx-auto max-w-5xl px-6">
                <!-- Feature panel -->
                <div class="relative overflow-hidden rounded-3xl bg-white p-10 shadow-xl shadow-black/10 md:p-14">
                    <!-- Decorative accent -->
                    <div class="mx-auto mb-6 h-1 w-16 rounded-full bg-gray-900" />

                    <!-- Heading -->
                    <div class="text-center">
                        <h2 class="text-3xl font-bold tracking-tight text-gray-900 sm:text-4xl" style="font-family: 'Playfair Display', serif;">
                            Plan Your Visit
                        </h2>
                        <p class="mt-3 text-base text-gray-600">
                            Everything you need for Sunday — in one place.
                        </p>
                    </div>

                    <!-- Expectation cards -->
                    <div class="mx-auto mt-10 grid max-w-4xl gap-4 sm:grid-cols-3">
                        <div class="rounded-2xl bg-gray-50 p-6 text-center shadow-sm">
                            <p class="font-semibold text-gray-900">Come as you are</p>
                            <p class="mt-2 text-sm text-gray-600">No dress code — you're welcome here.</p>
                        </div>

                        <div class="rounded-2xl bg-gray-50 p-6 text-center shadow-sm">
                            <p class="font-semibold text-gray-900">Warm welcome</p>
                            <p class="mt-2 text-sm text-gray-600">Friendly faces and a relaxed atmosphere.</p>
                        </div>

                        <div class="rounded-2xl bg-gray-50 p-6 text-center shadow-sm">
                            <p class="font-semibold text-gray-900">Biblical message</p>
                            <p class="mt-2 text-sm text-gray-600">Clear teaching you can apply to real life.</p>
                        </div>
                    </div>

                    <!-- Visit Info grid -->
                    <div class="mx-auto mt-12 max-w-3xl">
                        <div class="rounded-2xl border border-black/10 bg-white p-8">
                            <div class="grid gap-6 sm:grid-cols-3">
                                <div>
                                    <p class="text-xs font-semibold uppercase tracking-wider text-gray-600">Location</p>
                                    <p class="mt-2 text-sm leading-relaxed text-gray-900">
                                        New Beginnings AG<br />
                                        Grayson, KY
                                    </p>
                                </div>

                                <div>
                                    <p class="text-xs font-semibold uppercase tracking-wider text-gray-600">Service Times</p>
                                    <p class="mt-2 text-sm leading-relaxed text-gray-900">
                                        Sundays 10:30 AM<br />
                                        Wednesdays 7 PM
                                    </p>
                                </div>

                                <div>
                                    <p class="text-xs font-semibold uppercase tracking-wider text-gray-600">First Time Tip</p>
                                    <p class="mt-2 text-sm leading-relaxed text-gray-900">
                                        Arrive 10 min early &mdash; we'll help you find your way.
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- CTA -->
                    <div class="mt-10 flex flex-col items-center gap-3">
                        <Button size="lg" class="h-13 w-full rounded-full px-10 text-base font-semibold shadow-md shadow-primary/20 transition-all hover:shadow-lg hover:shadow-primary/30 sm:w-auto">
                            <MapPin class="mr-2 h-5 w-5" />
                            Get Directions
                        </Button>
                        <a href="#connect" class="text-sm font-medium text-gray-600 transition-colors hover:text-gray-900">
                            Contact Us
                        </a>
                    </div>
                </div>
            </div>
        </section>

        <!-- Stay Connected — Full-Width Closing Panel -->
        <section id="connect" class="relative isolate overflow-hidden">
            <!-- Background image -->
            <img
                src="https://images.unsplash.com/photo-1517457373958-b7bdd4587205?auto=format&fit=crop&w=1920&q=80"
                alt="Church community gathering together in fellowship"
                class="absolute inset-0 h-full w-full object-cover"
            />

            <!-- Dark gradient overlay for readability and intimacy -->
            <div class="absolute inset-0 bg-gradient-to-b from-black/65 via-black/55 to-black/65" />

            <!-- Content -->
            <div class="relative mx-auto max-w-4xl px-6 py-20 sm:py-28 text-center">
                <h2 class="text-3xl font-bold tracking-tight text-white sm:text-4xl" style="font-family: 'Playfair Display', serif;">
                    You Don't Have to Walk Alone
                </h2>
                
                <p class="mx-auto mt-6 max-w-2xl text-base leading-relaxed text-white/85 sm:text-lg">
                    Whether you're planning to visit soon or just want encouragement throughout the week, 
                    we'd love to stay connected and walk this journey of faith together.
                </p>

                <!-- Primary CTA -->
                <div class="mt-10">
                    <Button size="lg" variant="secondary" class="h-13 rounded-full px-10 text-base font-semibold shadow-2xl" as-child>
                        <a href="https://www.facebook.com/nbagGrayson" target="_blank" rel="noopener noreferrer">
                            <Facebook class="mr-2 h-5 w-5" />
                            Follow Us on Facebook
                        </a>
                    </Button>
                </div>

                <!-- Secondary link -->
                <div class="mt-4">
                    <a href="mailto:info@example.com" class="text-sm font-medium text-white/70 transition-colors hover:text-white">
                        Contact Us
                    </a>
                </div>
            </div>
        </section>

        <!-- Footer -->
        <footer class="border-t border-primary/10 bg-gradient-to-b from-background to-primary/[0.02] py-12">
            <div class="mx-auto max-w-6xl px-6">
                <div class="flex flex-col items-center gap-6 md:flex-row md:justify-between">
                    <div class="flex items-center gap-2.5">
                        <img src="/lion_icon_square.png" alt="New Beginnings AG Logo" class="h-8 w-8 rounded-sm" />
                        <div class="leading-tight">
                            <span class="block text-sm font-semibold tracking-tight">New Beginnings Assembly of God</span>
                            <span class="block text-[11px] text-muted-foreground">Grayson, Kentucky</span>
                        </div>
                    </div>

                    <nav class="flex flex-wrap items-center justify-center gap-x-6 gap-y-2 text-sm text-muted-foreground">
                        <a href="#about" class="transition-colors hover:text-primary">About</a>
                        <a href="#services" class="transition-colors hover:text-primary">Services</a>
                        <a href="#visit" class="transition-colors hover:text-primary">Visit</a>
                        <a href="https://www.facebook.com/nbagGrayson" target="_blank" rel="noopener noreferrer" class="transition-colors hover:text-primary">Facebook</a>
                    </nav>

                    <!-- Login/Register moved to footer -->
                    <div class="flex items-center gap-4 text-sm text-muted-foreground">
                        <Link :href="login()" class="transition-colors hover:text-primary">Log in</Link>
                        <Link v-if="canRegister" :href="register()" class="transition-colors hover:text-primary">Register</Link>
                    </div>
                </div>

                <Separator class="my-8 bg-primary/10" />

                <p class="text-center text-xs text-muted-foreground">
                    &copy; {{ new Date().getFullYear() }} New Beginnings Assembly of God. All rights reserved.
                </p>
        </div>
        </footer>
    </div>
</template>
