<script setup lang="ts">
import { Link } from '@inertiajs/vue3';
import { dashboard } from '@/routes';
import { Menu, ArrowRight, ChevronDown, Baby, Sparkles, Shield, Zap, Flame } from 'lucide-vue-next';
import { Button } from '@/components/ui/button';
import { Sheet, SheetContent, SheetTitle, SheetTrigger } from '@/components/ui/sheet';
import { ref, onMounted, onUnmounted } from 'vue';

const scrolled = ref(false);
const mobileOpen = ref(false);
const programsExpanded = ref(false);

const navItems = [
    { label: 'About', href: '#about' },
    { label: 'Services', href: '#services' },
    { label: 'Visit', href: '#visit' },
    { label: 'Connect', href: '#connect' },
];

const programs = [
    { label: 'Cubs of Judah', description: 'Children\'s Ministry', icon: Baby, color: 'text-amber-600 bg-amber-100' },
    { label: 'Soul Sisters', description: 'Women\'s Fellowship', icon: Sparkles, color: 'text-rose-600 bg-rose-100' },
    { label: 'Men of Valor', description: 'Men\'s Ministry', icon: Shield, color: 'text-sky-600 bg-sky-100' },
    { label: 'REWired', description: 'Teen Youth Group', icon: Zap, color: 'text-violet-600 bg-violet-100' },
    { label: 'Aftershock', description: 'Young Adults', icon: Flame, color: 'text-orange-600 bg-orange-100' },
];

function handleScroll(): void {
    scrolled.value = window.scrollY > 32;
}

onMounted(() => {
    if (typeof window !== 'undefined') {
        handleScroll();
        window.addEventListener('scroll', handleScroll, { passive: true });
    }
});

onUnmounted(() => {
    if (typeof window !== 'undefined') {
        window.removeEventListener('scroll', handleScroll);
    }
});
</script>

<template>
    <header
        :class="[
            'fixed inset-x-0 top-0 z-50 transition-all duration-300 ease-out',
            scrolled
                ? 'bg-white/90 shadow-sm backdrop-blur'
                : 'bg-transparent backdrop-blur-sm',
        ]"
    >
        <div class="mx-auto flex h-16 max-w-6xl items-center justify-between px-6">
            <!-- Logo lockup -->
            <a href="#" class="flex items-center gap-3">
                <img
                    src="/lion_icon_square.png"
                    alt="New Beginnings Assembly of God logo"
                    :class="[
                        'h-9 w-9 rounded-xl ring-1 transition-all duration-300 md:h-10 md:w-10',
                        scrolled ? 'ring-black/10' : 'ring-white/15',
                    ]"
                />
                <div class="leading-tight">
                    <span
                        :class="[
                            'block text-base font-semibold tracking-tight transition-colors duration-300 md:text-lg',
                            scrolled ? 'text-gray-900' : 'text-white',
                        ]"
                    >
                        New Beginnings
                    </span>
                    <span
                        :class="[
                            'hidden text-[11px] transition-colors duration-300 sm:block',
                            scrolled ? 'text-gray-500' : 'text-white/60',
                        ]"
                    >
                        Assembly of God · Grayson, KY
                    </span>
                </div>
            </a>

            <!-- Desktop nav (centered) -->
            <nav class="hidden items-center gap-8 md:flex">
                <!-- About -->
                <a
                    :href="navItems[0].href"
                    :class="[
                        'text-sm font-medium transition-all duration-200 underline-offset-8 decoration-2 hover:underline',
                        scrolled
                            ? 'text-gray-600 decoration-gray-900/20 hover:text-gray-900'
                            : 'text-white/80 decoration-white/40 hover:text-white',
                    ]"
                >
                    {{ navItems[0].label }}
                </a>

                <!-- Services -->
                <a
                    :href="navItems[1].href"
                    :class="[
                        'text-sm font-medium transition-all duration-200 underline-offset-8 decoration-2 hover:underline',
                        scrolled
                            ? 'text-gray-600 decoration-gray-900/20 hover:text-gray-900'
                            : 'text-white/80 decoration-white/40 hover:text-white',
                    ]"
                >
                    {{ navItems[1].label }}
                </a>

                <!-- Programs dropdown -->
                <div class="group relative">
                    <a
                        href="#programs"
                        :class="[
                            'inline-flex items-center gap-1 text-sm font-medium transition-all duration-200 underline-offset-8 decoration-2 hover:underline',
                            scrolled
                                ? 'text-gray-600 decoration-gray-900/20 hover:text-gray-900'
                                : 'text-white/80 decoration-white/40 hover:text-white',
                        ]"
                    >
                        Programs
                        <ChevronDown class="h-3.5 w-3.5 opacity-60 transition-transform duration-200 group-hover:rotate-180" />
                    </a>

                    <!-- Dropdown panel -->
                    <div class="pointer-events-none absolute left-1/2 top-full pt-3 opacity-0 transition-all duration-200 group-hover:pointer-events-auto group-hover:opacity-100">
                        <div class="-translate-x-1/2 rounded-2xl border border-black/5 bg-white p-2 shadow-xl shadow-black/10" style="min-width: 280px;">
                            <a
                                v-for="program in programs"
                                :key="program.label"
                                href="#programs"
                                class="flex items-center gap-3 rounded-xl px-3 py-2.5 transition-colors hover:bg-gray-50"
                            >
                                <div :class="['flex h-9 w-9 shrink-0 items-center justify-center rounded-lg', program.color]">
                                    <component :is="program.icon" class="h-4.5 w-4.5" />
                                </div>
                                <div>
                                    <p class="text-sm font-semibold text-gray-900">{{ program.label }}</p>
                                    <p class="text-xs text-gray-500">{{ program.description }}</p>
                                </div>
                            </a>
                        </div>
                    </div>
                </div>

                <!-- Visit -->
                <a
                    :href="navItems[2].href"
                    :class="[
                        'text-sm font-medium transition-all duration-200 underline-offset-8 decoration-2 hover:underline',
                        scrolled
                            ? 'text-gray-600 decoration-gray-900/20 hover:text-gray-900'
                            : 'text-white/80 decoration-white/40 hover:text-white',
                    ]"
                >
                    {{ navItems[2].label }}
                </a>

                <!-- Connect -->
                <a
                    :href="navItems[3].href"
                    :class="[
                        'text-sm font-medium transition-all duration-200 underline-offset-8 decoration-2 hover:underline',
                        scrolled
                            ? 'text-gray-600 decoration-gray-900/20 hover:text-gray-900'
                            : 'text-white/80 decoration-white/40 hover:text-white',
                    ]"
                >
                    {{ navItems[3].label }}
                </a>
            </nav>

            <!-- Right side -->
            <div class="flex items-center gap-3">
                <!-- Auth link (if logged in) -->
                <Link v-if="$page.props.auth.user" :href="dashboard()">
                    <Button
                        size="sm"
                        :class="[
                            'h-8 rounded-full px-4 text-xs font-medium transition-all duration-300',
                            scrolled
                                ? 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                                : 'bg-white/10 text-white/80 backdrop-blur-sm hover:bg-white/15',
                        ]"
                    >
                        Dashboard
                    </Button>
                </Link>

                <!-- CTA button -->
                <a v-if="!$page.props.auth.user" href="#visit" class="hidden sm:block">
                    <button
                        :class="[
                            'inline-flex items-center gap-2 rounded-2xl px-5 py-2.5 text-sm font-semibold transition-all duration-300 hover:-translate-y-0.5',
                            scrolled
                                ? 'bg-gray-900 text-white shadow-lg shadow-black/10 hover:bg-gray-800'
                                : 'bg-white text-gray-900 shadow-lg shadow-black/25 hover:shadow-xl',
                        ]"
                    >
                        Plan Your Visit
                    </button>
                </a>

                <!-- Mobile menu trigger -->
                <Sheet v-model:open="mobileOpen">
                    <SheetTrigger as-child>
                        <button
                            :class="[
                                'inline-flex h-10 w-10 items-center justify-center rounded-xl transition-colors duration-300 md:hidden',
                                scrolled
                                    ? 'text-gray-700 hover:bg-gray-100'
                                    : 'text-white/90 hover:bg-white/10',
                            ]"
                            aria-label="Open menu"
                        >
                            <Menu class="h-5 w-5" />
                        </button>
                    </SheetTrigger>

                    <SheetContent side="right" class="w-80 bg-white p-0">
                        <SheetTitle class="sr-only">Navigation Menu</SheetTitle>

                        <!-- Mobile menu content -->
                        <div class="flex h-full flex-col">
                            <!-- Logo in menu -->
                            <div class="flex items-center gap-3 border-b px-6 py-5">
                                <img
                                    src="/lion_icon_square.png"
                                    alt="New Beginnings Assembly of God logo"
                                    class="h-10 w-10 rounded-xl ring-1 ring-black/10"
                                />
                                <div class="leading-tight">
                                    <span class="block text-base font-semibold tracking-tight text-gray-900">
                                        New Beginnings
                                    </span>
                                    <span class="block text-[11px] text-gray-500">
                                        Assembly of God · Grayson, KY
                                    </span>
                                </div>
                            </div>

                            <!-- Nav links -->
                            <nav class="flex-1 overflow-y-auto px-4 py-4">
                                <a
                                    v-for="item in navItems.slice(0, 2)"
                                    :key="item.label"
                                    :href="item.href"
                                    class="flex items-center rounded-xl px-4 py-3 text-[15px] font-medium text-gray-700 transition-colors hover:bg-gray-50 hover:text-gray-900"
                                    @click="mobileOpen = false"
                                >
                                    {{ item.label }}
                                </a>

                                <!-- Programs expandable -->
                                <button
                                    class="flex w-full items-center justify-between rounded-xl px-4 py-3 text-[15px] font-medium text-gray-700 transition-colors hover:bg-gray-50 hover:text-gray-900"
                                    @click="programsExpanded = !programsExpanded"
                                >
                                    Programs
                                    <ChevronDown
                                        :class="[
                                            'h-4 w-4 text-gray-400 transition-transform duration-200',
                                            programsExpanded ? 'rotate-180' : '',
                                        ]"
                                    />
                                </button>

                                <!-- Programs sub-items -->
                                <div
                                    :class="[
                                        'overflow-hidden transition-all duration-200',
                                        programsExpanded ? 'max-h-96 opacity-100' : 'max-h-0 opacity-0',
                                    ]"
                                >
                                    <div class="pb-1 pl-2">
                                        <a
                                            v-for="program in programs"
                                            :key="program.label"
                                            href="#programs"
                                            class="flex items-center gap-3 rounded-xl px-3 py-2.5 transition-colors hover:bg-gray-50"
                                            @click="mobileOpen = false"
                                        >
                                            <div :class="['flex h-8 w-8 shrink-0 items-center justify-center rounded-lg', program.color]">
                                                <component :is="program.icon" class="h-4 w-4" />
                                            </div>
                                            <div>
                                                <p class="text-sm font-medium text-gray-900">{{ program.label }}</p>
                                                <p class="text-[11px] text-gray-500">{{ program.description }}</p>
                                            </div>
                                        </a>
                                    </div>
                                </div>

                                <a
                                    v-for="item in navItems.slice(2)"
                                    :key="item.label"
                                    :href="item.href"
                                    class="flex items-center rounded-xl px-4 py-3 text-[15px] font-medium text-gray-700 transition-colors hover:bg-gray-50 hover:text-gray-900"
                                    @click="mobileOpen = false"
                                >
                                    {{ item.label }}
                                </a>
                            </nav>

                            <!-- Mobile CTA -->
                            <div class="border-t px-6 py-5">
                                <a href="#visit" @click="mobileOpen = false">
                                    <button
                                        class="flex w-full items-center justify-center gap-2 rounded-2xl bg-gray-900 px-6 py-3.5 text-sm font-semibold text-white shadow-lg shadow-black/10 transition-colors hover:bg-gray-800"
                                    >
                                        Plan Your Visit
                                        <ArrowRight class="h-4 w-4" />
                                    </button>
                                </a>
                            </div>
                        </div>
                    </SheetContent>
                </Sheet>
            </div>
        </div>
    </header>
</template>
