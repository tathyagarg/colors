<script lang="ts">
  type Scheme = "complementary" | "analogous" | "split" | "triadic";
  type HueItem = {
    name: string;
    hue: number;
    type: "Primary" | "Secondary" | "Tertiary";
  };
  const hues: HueItem[] = [
    { name: "Red", hue: 0, type: "Primary" },
    { name: "Red-orange", hue: 30, type: "Tertiary" },
    { name: "Orange", hue: 30, type: "Secondary" },
    { name: "Yellow-orange", hue: 45, type: "Tertiary" },
    { name: "Yellow", hue: 60, type: "Primary" },
    { name: "Yellow-green", hue: 90, type: "Tertiary" },
    { name: "Green", hue: 120, type: "Secondary" },
    { name: "Blue-green", hue: 165, type: "Tertiary" },
    { name: "Blue", hue: 210, type: "Primary" },
    { name: "Blue-violet", hue: 240, type: "Tertiary" },
    { name: "Violet", hue: 270, type: "Secondary" },
    { name: "Red-violet", hue: 315, type: "Tertiary" },
  ];
  const schemeOptions: { id: Scheme; title: string; subtitle: string }[] = [
    {
      id: "complementary",
      title: "Complementary",
      subtitle: "Opposites create contrast.",
    },
    {
      id: "analogous",
      title: "Analogous",
      subtitle: "Neighbors create harmony.",
    },
    {
      id: "split",
      title: "Split complementary",
      subtitle: "One base, two near-opposites.",
    },
    { id: "triadic", title: "Triadic", subtitle: "Three evenly spaced hues." },
  ];

  let selectedHue = $state(210);
  let saturation = $state(76);
  let tint = $state(0);
  let selectedIndex = $state(8);
  let activeScheme = $state<Scheme>("complementary");
  let temperature = $state<"warm" | "cool">("warm");
  let copied = $state("");
  let dragging = false;

  function hsl(h: number, s = saturation, l = 50) {
    return `hsl(${((h % 360) + 360) % 360} ${s}% ${l}%)`;
  }
  function hex(h: number, s: number, l: number) {
    h = ((h % 360) + 360) % 360;
    s /= 100;
    l /= 100;
    const k = (n: number) => (n + h / 30) % 12;
    const a = s * Math.min(l, 1 - l);
    const f = (n: number) => {
      const value =
        l - a * Math.max(-1, Math.min(k(n) - 3, Math.min(9 - k(n), 1)));
      return Math.round(255 * value)
        .toString(16)
        .padStart(2, "0");
    };
    return `#${f(0)}${f(8)}${f(4)}`.toUpperCase();
  }
  let tintLightness = $derived(50 + tint * 0.42);
  let mainColor = $derived(hsl(selectedHue, saturation, tintLightness));
  let mainHex = $derived(hex(selectedHue, saturation, tintLightness));
  let activeName = $derived(hues[selectedIndex].name);
  let palette = $derived.by(() => {
    const rules: Record<
      Scheme,
      { label: string; offset: number; role: string }[]
    > = {
      complementary: [
        { label: "Base", offset: 0, role: "Main color" },
        { label: "Complement", offset: 180, role: "Contrast color" },
      ],
      analogous: [
        { label: "Neighbor A", offset: -30, role: "Supporting color" },
        { label: "Base", offset: 0, role: "Main color" },
        { label: "Neighbor B", offset: 30, role: "Supporting color" },
      ],
      split: [
        { label: "Base", offset: 0, role: "Main color" },
        { label: "Split A", offset: 150, role: "Accent color" },
        { label: "Split B", offset: 210, role: "Accent color" },
      ],
      triadic: [
        { label: "Base", offset: 0, role: "Main color" },
        { label: "Triad A", offset: 120, role: "Accent color" },
        { label: "Triad B", offset: 240, role: "Accent color" },
      ],
    };
    return rules[activeScheme].map((item) => ({
      ...item,
      hue: (selectedHue + item.offset + 360) % 360,
      color: hsl(selectedHue + item.offset, saturation, tintLightness),
      hex: hex(selectedHue + item.offset, saturation, tintLightness),
    }));
  });
  function chooseHue(index: number) {
    selectedIndex = index;
    selectedHue = hues[index].hue;
    saturation = 76;
    tint = 0;
  }
  function nearestHue(hue: number) {
    return hues.reduce((best, item, index) => {
      const d = Math.abs(((item.hue - hue + 540) % 360) - 180);
      const bestD = Math.abs(((hues[best].hue - hue + 540) % 360) - 180);
      return d < bestD ? index : best;
    }, 0);
  }
  function pointerHue(event: PointerEvent, element: SVGSVGElement) {
    const rect = element.getBoundingClientRect();
    const x = event.clientX - rect.left - rect.width / 2;
    const y = event.clientY - rect.top - rect.height / 2;
    selectedHue = ((Math.atan2(y, x) * 180) / Math.PI + 450) % 360;
    selectedIndex = nearestHue(selectedHue);
  }
  async function copyHex(value: string) {
    try {
      await navigator.clipboard.writeText(value);
      copied = value;
      setTimeout(() => (copied = ""), 1300);
    } catch {
      copied = "Copy unavailable";
      setTimeout(() => (copied = ""), 1300);
    }
  }
  const warmColors = [
    { name: "Red", meaning: "Energy · urgency", color: "#E84C3D" },
    { name: "Orange", meaning: "Playfulness · warmth", color: "#F28B30" },
    { name: "Yellow", meaning: "Optimism · attention", color: "#E8C547" },
  ];
  const coolColors = [
    { name: "Blue", meaning: "Trust · calm", color: "#3878C7" },
    { name: "Teal", meaning: "Balance · clarity", color: "#329B99" },
    { name: "Violet", meaning: "Imagination · mystery", color: "#8170BD" },
  ];
</script>

<svelte:head>
  <title>Color Theory — A Visual Field Guide</title>
  <meta
    name="description"
    content="Learn color theory visually with an interactive artist's color wheel, color properties, temperature and harmony palettes."
  />
</svelte:head>

<header
  class="sticky top-0 z-50 border-b border-neutral-200 bg-white/95 backdrop-blur"
>
  <div
    class="mx-auto flex h-[68px] max-w-7xl items-center justify-between gap-4 px-5 md:px-10"
  >
    <a
      href="#top"
      class="flex shrink-0 items-center gap-2 text-[11px] font-extrabold tracking-[.12em]"
    >
      <span class="flex -rotate-12 gap-[2px]" aria-hidden="true"
        ><i class="h-4 w-1.5 rounded-full bg-red-500"></i><i
          class="h-4 w-1.5 rounded-full bg-amber-400"
        ></i><i class="h-4 w-1.5 rounded-full bg-blue-600"></i></span
      >
      COLOR THEORY
      <span class="hidden font-medium text-neutral-400 sm:inline"
        >/ FIELD GUIDE</span
      >
    </a>
    <nav
      aria-label="Main navigation"
      class="hidden items-center gap-7 text-xs text-neutral-600 md:flex"
    >
      <a class="transition hover:text-blue-700" href="#wheel">The wheel</a>
      <a class="transition hover:text-blue-700" href="#properties">Properties</a
      >
      <a class="transition hover:text-blue-700" href="#temperature"
        >Temperature</a
      >
      <a class="transition hover:text-blue-700" href="#harmony">Harmony</a>
    </nav>
  </div>
</header>

<main id="top">
  <section
    class="mx-auto grid max-w-7xl items-center gap-10 px-5 py-14 md:grid-cols-2 md:px-10 md:py-20"
  >
    <div>
      <p
        class="mb-6 flex items-center gap-2 text-[9px] font-bold tracking-[.18em] text-neutral-500"
      >
        <span class="h-px w-6 bg-neutral-500"></span> A VISUAL FIELD GUIDE / 01—04
      </p>
      <h1
        class="text-6xl font-medium leading-[.94] tracking-[-.075em] sm:text-7xl lg:text-8xl"
      >
        Color is a<br />way of
        <span class="font-normal text-neutral-500">seeing.</span>
      </h1>
      <p class="mt-7 max-w-md text-sm leading-7 text-neutral-600">
        A practical introduction to how colors work together. No theory-heavy
        lectures — just a wheel, a few simple rules, and room to experiment.
      </p>
      <a
        href="#wheel"
        class="mt-7 inline-flex items-center gap-6 border-b border-neutral-400 pb-2 text-xs font-semibold transition hover:text-blue-700"
        >Start with the wheel <span aria-hidden="true">↓</span></a
      >
      <div
        class="mt-12 flex items-center gap-3 text-[8px] tracking-[.16em] text-neutral-400"
      >
        <span class="font-bold text-neutral-900">01</span><span
          class="h-px w-10 bg-neutral-300"
        ></span> COLOR, MADE VISIBLE
      </div>
    </div>
    <div
      class="relative flex min-h-[320px] items-center justify-center overflow-hidden bg-neutral-50 sm:min-h-[410px]"
      aria-label="Abstract overlapping color composition"
    >
      <div
        class="absolute h-[80%] aspect-square rounded-full border border-neutral-200"
      ></div>
      <div
        class="absolute h-[61%] aspect-square rounded-full border border-neutral-200"
      ></div>
      <div
        class="absolute left-[20%] top-[25%] h-40 w-40 rounded-full bg-red-500/90 mix-blend-multiply sm:h-48 sm:w-48"
      ></div>
      <div
        class="absolute left-[48%] top-[25%] h-40 w-40 rounded-full bg-yellow-300/95 mix-blend-multiply sm:h-48 sm:w-48"
      ></div>
      <div
        class="absolute left-[47%] top-[48%] h-40 w-40 rounded-full bg-blue-600/90 mix-blend-multiply sm:h-48 sm:w-48"
      ></div>
      <div
        class="absolute left-[25%] top-[54%] h-24 w-24 rounded-full bg-violet-300/90 mix-blend-multiply"
      ></div>
      <div
        class="absolute left-[54%] top-[23%] grid h-24 w-24 -translate-x-1/2 items-center justify-items-center rounded-full bg-white text-center text-[8px] font-bold leading-4 tracking-widest shadow-sm"
      >
        COLOR<br />IN CONTEXT<br /><span class="text-base font-normal">↗</span>
      </div>
      <span
        class="absolute left-5 top-5 text-[8px] tracking-[.14em] text-neutral-500"
        >HUE / 360°</span
      ><span
        class="absolute bottom-5 right-5 text-[8px] tracking-[.14em] text-neutral-500"
        >A WORLD OF COMBINATIONS</span
      >
    </div>
  </section>

  <section
    id="wheel"
    class="scroll-mt-20 border-t border-neutral-200 bg-neutral-50 py-16 md:py-20"
  >
    <div class="mx-auto max-w-7xl px-5 md:px-10">
      <div
        class="mb-10 flex flex-col justify-between gap-5 md:flex-row md:items-end"
      >
        <div>
          <p
            class="mb-3 text-[9px] font-bold tracking-[.18em] text-neutral-500"
          >
            01 / THE FOUNDATION
          </p>
          <h2 class="text-4xl font-medium tracking-[-.055em] sm:text-5xl">
            Meet the color wheel.
          </h2>
        </div>
        <p class="max-w-sm text-sm leading-7 text-neutral-600">
          Twelve hues arranged in a circle. Their positions help you predict
          which colors contrast, blend, or balance.
        </p>
      </div>
      <div class="grid items-center gap-9 lg:grid-cols-2 lg:gap-16">
        <div class="flex flex-col items-center">
          <svg
            viewBox="0 0 400 400"
            role="group"
            aria-labelledby="wheel-title wheel-description"
            class="w-full max-w-[400px] touch-none overflow-visible"
            onpointerdown={(event) => {
              dragging = true;
              event.currentTarget.setPointerCapture(event.pointerId);
              pointerHue(event, event.currentTarget);
            }}
            onpointermove={(event) => {
              if (dragging) pointerHue(event, event.currentTarget);
            }}
            onpointerup={() => (dragging = false)}
            onpointercancel={() => (dragging = false)}
          >
            <title id="wheel-title">Interactive artist's color wheel</title
            ><desc id="wheel-description"
              >Choose a labeled segment with a click or keyboard, or drag around
              the wheel to change the hue.</desc
            >
            {#each hues as item, i}
              {@const start = ((i * 30 - 105) * Math.PI) / 180}
              {@const end = (((i + 1) * 30 - 105) * Math.PI) / 180}
              {@const point = (r: number, a: number) =>
                `${200 + r * Math.cos(a)},${200 + r * Math.sin(a)}`}
              <path
                d={`M ${point(82, start)} L ${point(172, start)} A 172 172 0 0 1 ${point(172, end)} L ${point(82, end)} A 82 82 0 0 0 ${point(82, start)} Z`}
                fill={hsl(item.hue, 76, 52)}
                stroke="white"
                stroke-width="3"
                tabindex="0"
                role="button"
                aria-label={`Select ${item.name}, ${item.type} color`}
                class="cursor-pointer outline-none transition hover:brightness-110 focus-visible:stroke-neutral-900 focus-visible:stroke-[5]"
                onclick={() => chooseHue(i)}
                onkeydown={(e) => {
                  if (e.key === "Enter" || e.key === " ") {
                    e.preventDefault();
                    chooseHue(i);
                  }
                }}
              />
            {/each}
            <circle
              cx="200"
              cy="200"
              r="75"
              fill="white"
              pointer-events="none"
            />
            <text
              x="200"
              y="190"
              text-anchor="middle"
              fill="#888"
              font-size="9"
              font-weight="700"
              letter-spacing="1.5"
              pointer-events="none">YOUR HUE</text
            >
            <text
              x="200"
              y="220"
              text-anchor="middle"
              fill="#171717"
              font-size="24"
              font-weight="600"
              pointer-events="none">{Math.round(selectedHue)}°</text
            >
            <circle
              cx="200"
              cy="200"
              r="184"
              fill="none"
              stroke="#e5e5e5"
              stroke-width="1"
              pointer-events="none"
            />
            <rect
              x="0"
              y="0"
              width="400"
              height="400"
              fill="transparent"
              pointer-events="none"
            />
          </svg>
          <p class="mt-4 text-[10px] text-neutral-500">
            Select a segment or drag the wheel to explore
          </p>
          <div class="mt-4 flex flex-wrap justify-center gap-2">
            {#each hues as item, i}
              <button
                aria-label={`Choose ${item.name}`}
                title={item.name}
                class={`h-5 w-5 rounded-full border-2 transition hover:scale-110 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 ${selectedIndex === i ? "border-neutral-900 ring-2 ring-white" : "border-white"}`}
                style={`background:${hsl(item.hue, 76, 52)}`}
                onclick={() => chooseHue(i)}
              ></button>
            {/each}
          </div>
        </div>
        <div class="border border-neutral-200 bg-white p-5 sm:p-7">
          <div
            class="flex justify-between text-[8px] font-bold tracking-[.15em] text-neutral-500"
          >
            <span>SELECTED COLOR</span><span>RYB / 12 HUES</span>
          </div>
          <div class="flex items-center gap-3 border-b border-neutral-200 py-6">
            <div
              class="h-11 w-11 shrink-0"
              style={`background:${hsl(hues[selectedIndex].hue, 76, 52)}`}
            ></div>
            <div>
              <h3 class="text-lg font-semibold tracking-tight">{activeName}</h3>
              <p class="mt-1 text-[10px] text-neutral-500">
                {hues[selectedIndex].type} color
              </p>
            </div>
            <button
              class="ml-auto border border-neutral-200 px-2.5 py-2 text-[10px] transition hover:border-neutral-700"
              onclick={() => copyHex(hex(hues[selectedIndex].hue, 76, 52))}
              >{hex(hues[selectedIndex].hue, 76, 52)} ⧉</button
            >
          </div>
          <div class="divide-y divide-neutral-100">
            <button
              class="grid w-full grid-cols-[22px_20px_1fr_12px] items-center gap-3 py-4 text-left hover:bg-neutral-50"
              onclick={() => chooseHue(0)}
              ><span class="text-[9px] text-neutral-400">01</span><span
                class="h-5 w-5 rounded-full bg-red-500"
              ></span><span
                ><b class="block text-[11px]">Primary</b><small
                  class="mt-1 block text-[10px] text-neutral-500"
                  >Red · Yellow · Blue</small
                ></span
              ><span class="text-xs text-neutral-400">↗</span></button
            >
            <button
              class="grid w-full grid-cols-[22px_20px_1fr_12px] items-center gap-3 py-4 text-left hover:bg-neutral-50"
              onclick={() => chooseHue(2)}
              ><span class="text-[9px] text-neutral-400">02</span><span
                class="h-5 w-5 rounded-full bg-orange-500"
              ></span><span
                ><b class="block text-[11px]">Secondary</b><small
                  class="mt-1 block text-[10px] text-neutral-500"
                  >Orange · Green · Violet</small
                ></span
              ><span class="text-xs text-neutral-400">↗</span></button
            >
            <button
              class="grid w-full grid-cols-[22px_20px_1fr_12px] items-center gap-3 py-4 text-left hover:bg-neutral-50"
              onclick={() => chooseHue(1)}
              ><span class="text-[9px] text-neutral-400">03</span><span
                class="h-5 w-5 rounded-full bg-rose-400"
              ></span><span
                ><b class="block text-[11px]">Tertiary</b><small
                  class="mt-1 block text-[10px] text-neutral-500"
                  >Mix a primary with a neighboring secondary</small
                ></span
              ><span class="text-xs text-neutral-400">↗</span></button
            >
          </div>
          <p class="mt-4 text-[10px] leading-6 text-neutral-500">
            The traditional artist’s wheel uses red, yellow, and blue as
            primaries. Digital screens use a different model: RGB.
          </p>
        </div>
      </div>
      <div
        class="mt-8 flex items-start gap-3 border-t border-neutral-200 pt-5 text-xs leading-6 text-neutral-600"
      >
        <span class="text-lg">↳</span>
        <p>
          <b class="text-neutral-900">Notice the neighbors.</b> Colors close together
          share a similar feel. Colors across the wheel create stronger contrast.
        </p>
        <span
          class="ml-auto shrink-0 text-[8px] tracking-widest text-neutral-400"
          >FIG. 01</span
        >
      </div>
    </div>
  </section>

  <section
    id="properties"
    class="scroll-mt-20 border-t border-neutral-200 py-16 md:py-20"
  >
    <div class="mx-auto max-w-7xl px-5 md:px-10">
      <div
        class="mb-10 flex flex-col justify-between gap-5 md:flex-row md:items-end"
      >
        <div>
          <p
            class="mb-3 text-[9px] font-bold tracking-[.18em] text-neutral-500"
          >
            02 / THREE CONTROLS
          </p>
          <h2 class="text-4xl font-medium tracking-[-.055em] sm:text-5xl">
            One color. Many possibilities.
          </h2>
        </div>
        <p class="max-w-sm text-sm leading-7 text-neutral-600">
          A color is more than its name. Change its hue, saturation, or tint to
          change how it feels and where it works.
        </p>
      </div>
      <div class="grid items-center gap-10 lg:grid-cols-2 lg:gap-16">
        <div class="space-y-7">
          <div class="border-b border-neutral-200 pb-6">
            <div class="mb-5 grid grid-cols-[28px_1fr_auto] items-start gap-3">
              <span
                class="grid h-6 w-6 place-items-center border border-neutral-300 text-[10px] text-neutral-500"
                >A</span
              >
              <div>
                <h3 class="text-sm font-semibold">Hue</h3>
                <p class="mt-1 text-[11px] leading-5 text-neutral-500">
                  The color family — red, green, blue, and everything between.
                </p>
              </div>
              <output class="text-xs font-semibold"
                >{Math.round(selectedHue)}°</output
              >
            </div>
            <input
              aria-label="Hue"
              class="h-1 w-full cursor-pointer accent-neutral-900"
              type="range"
              min="0"
              max="359"
              bind:value={selectedHue}
            />
            <div class="mt-3 flex justify-between text-[9px] text-neutral-400">
              <span>Red</span><span>Yellow</span><span>Green</span><span
                >Blue</span
              ><span>Red</span>
            </div>
          </div>
          <div class="border-b border-neutral-200 pb-6">
            <div class="mb-5 grid grid-cols-[28px_1fr_auto] items-start gap-3">
              <span
                class="grid h-6 w-6 place-items-center border border-neutral-300 text-[10px] text-neutral-500"
                >B</span
              >
              <div>
                <h3 class="text-sm font-semibold">Saturation</h3>
                <p class="mt-1 text-[11px] leading-5 text-neutral-500">
                  Color intensity — vivid at one end, muted at the other.
                </p>
              </div>
              <output class="text-xs font-semibold">{saturation}%</output>
            </div>
            <input
              aria-label="Saturation"
              class="h-1 w-full cursor-pointer accent-neutral-900"
              type="range"
              min="0"
              max="100"
              bind:value={saturation}
            />
            <div class="mt-3 flex justify-between text-[9px] text-neutral-400">
              <span>Muted</span><span>Vivid</span>
            </div>
          </div>
          <div class="border-b border-neutral-200 pb-6">
            <div class="mb-5 grid grid-cols-[28px_1fr_auto] items-start gap-3">
              <span
                class="grid h-6 w-6 place-items-center border border-neutral-300 text-[10px] text-neutral-500"
                >C</span
              >
              <div>
                <h3 class="text-sm font-semibold">Tint</h3>
                <p class="mt-1 text-[11px] leading-5 text-neutral-500">
                  Add white to a hue to create a lighter, softer version.
                </p>
              </div>
              <output class="text-xs font-semibold">{tint}%</output>
            </div>
            <input
              aria-label="Tint"
              class="h-1 w-full cursor-pointer accent-neutral-900"
              type="range"
              min="0"
              max="100"
              bind:value={tint}
            />
            <div class="mt-3 flex justify-between text-[9px] text-neutral-400">
              <span>Pure hue</span><span>More white</span>
            </div>
          </div>
          <p class="bg-neutral-50 p-4 text-[10px] leading-6 text-neutral-600">
            <b class="text-neutral-900">Try this:</b> Keep the hue fixed and move
            one slider at a time. Notice how the same color can become quieter, louder,
            or lighter.
          </p>
        </div>
        <div class="border border-neutral-200 bg-neutral-50 p-5 sm:p-6">
          <div
            class="flex justify-between text-[8px] font-bold tracking-[.14em] text-neutral-500"
          >
            <span>LIVE PREVIEW</span><span>HSL / COLOR VALUES</span>
          </div>
          <div
            class="mt-5 h-52 transition-colors duration-150"
            style={`background:${mainColor}`}
            role="img"
            aria-label={`Current color preview, ${mainHex}`}
          ></div>
          <div class="mt-5 flex items-center justify-between">
            <div>
              <span
                class="block text-[8px] font-bold tracking-widest text-neutral-500"
                >YOUR COLOR</span
              ><strong class="mt-1 block text-xl font-semibold tracking-tight"
                >{mainHex}</strong
              >
            </div>
            <button
              class="border border-neutral-200 bg-white px-3 py-2 text-[10px] hover:border-neutral-700"
              onclick={() => copyHex(mainHex)}
              >{copied === mainHex ? "Copied ✓" : "Copy HEX ⧉"}</button
            >
          </div>
          <div
            class="mt-5 grid grid-cols-6 gap-1.5"
            aria-label="Tint comparison"
          >
            {#each [0, 20, 40, 60, 80, 100] as amount}
              <button
                class="relative h-11 transition hover:-translate-y-0.5 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2"
                style={`background:${hsl(selectedHue, saturation, 50 + amount * 0.42)}`}
                aria-label={`Set tint to ${amount}%`}
                onclick={() => (tint = amount)}
                ><span
                  class="absolute inset-x-0 bottom-1 text-center text-[8px] text-black/70"
                  >{amount}</span
                ></button
              >
            {/each}
          </div>
          <p class="mt-3 text-[10px] text-neutral-500">
            Select a tint above or adjust the controls to make your own color.
          </p>
        </div>
      </div>
      <div
        class="mt-12 grid grid-cols-3 gap-4 border-t border-neutral-200 pt-6"
      >
        <div>
          <span class="text-[9px] text-neutral-400">01</span>
          <h3 class="mt-2 text-xs font-semibold">Hue</h3>
          <p class="mt-1 text-[10px] text-neutral-500">Which color it is.</p>
        </div>
        <div>
          <span class="text-[9px] text-neutral-400">02</span>
          <h3 class="mt-2 text-xs font-semibold">Saturation</h3>
          <p class="mt-1 text-[10px] text-neutral-500">How intense it looks.</p>
        </div>
        <div>
          <span class="text-[9px] text-neutral-400">03</span>
          <h3 class="mt-2 text-xs font-semibold">Tint</h3>
          <p class="mt-1 text-[10px] text-neutral-500">
            How much white is added.
          </p>
        </div>
      </div>
    </div>
  </section>

  <section
    id="temperature"
    class="scroll-mt-20 border-t border-neutral-200 bg-neutral-50 py-16 md:py-20"
  >
    <div class="mx-auto max-w-7xl px-5 md:px-10">
      <div
        class="mb-10 flex flex-col justify-between gap-5 md:flex-row md:items-end"
      >
        <div>
          <p
            class="mb-3 text-[9px] font-bold tracking-[.18em] text-neutral-500"
          >
            03 / COLOR TEMPERATURE
          </p>
          <h2 class="text-4xl font-medium tracking-[-.055em] sm:text-5xl">
            Feel the temperature.
          </h2>
        </div>
        <p class="max-w-sm text-sm leading-7 text-neutral-600">
          Colors can suggest warmth or coolness. It’s an association, not a rule
          — context changes the feeling.
        </p>
      </div>
      <div
        class="mb-5 flex border-b border-neutral-300"
        role="group"
        aria-label="Choose color temperature"
      >
        <button
          class={`-mb-px flex items-center gap-2 border-b-2 px-2 py-4 pr-6 text-xs ${temperature === "warm" ? "border-neutral-900 text-neutral-900" : "border-transparent text-neutral-500"}`}
          aria-pressed={temperature === "warm"}
          onclick={() => (temperature = "warm")}
          ><span class="text-lg text-orange-600">◒</span> Warm colors
          <span class="ml-3 text-[9px] text-neutral-400">01</span></button
        >
        <button
          class={`-mb-px flex items-center gap-2 border-b-2 px-2 py-4 text-xs ${temperature === "cool" ? "border-neutral-900 text-neutral-900" : "border-transparent text-neutral-500"}`}
          aria-pressed={temperature === "cool"}
          onclick={() => (temperature = "cool")}
          ><span class="text-lg text-blue-600">◒</span> Cool colors
          <span class="ml-3 text-[9px] text-neutral-400">02</span></button
        >
      </div>
      <div class="grid bg-white md:grid-cols-2">
        <div
          class={`relative min-h-[300px] overflow-hidden transition-colors duration-300 sm:min-h-[380px] ${temperature === "warm" ? "bg-orange-100" : "bg-sky-100"}`}
          role="img"
          aria-label={`${temperature} abstract color composition`}
        >
          {#if temperature === "warm"}
            <div
              class="absolute -left-[15%] -top-[10%] h-[110%] w-[62%] rounded-full bg-red-500"
            ></div>
            <div
              class="absolute left-[30%] top-[25%] h-[85%] w-[58%] rotate-[-20deg] rounded-[48%_52%_0_0] bg-orange-400"
            ></div>
            <div
              class="absolute left-[52%] top-[5%] h-[35%] w-[35%] rounded-full bg-yellow-300"
            ></div>
            <div
              class="absolute left-[58%] top-[59%] h-[28%] w-[28%] rounded-full bg-red-800"
            ></div>
          {:else}
            <div
              class="absolute -left-[15%] -top-[10%] h-[110%] w-[62%] rounded-full bg-blue-600"
            ></div>
            <div
              class="absolute left-[30%] top-[25%] h-[85%] w-[58%] rotate-[-20deg] rounded-[48%_52%_0_0] bg-teal-400"
            ></div>
            <div
              class="absolute left-[52%] top-[5%] h-[35%] w-[35%] rounded-full bg-blue-200"
            ></div>
            <div
              class="absolute left-[58%] top-[59%] h-[28%] w-[28%] rounded-full bg-indigo-900"
            ></div>
          {/if}
          <span
            class="absolute bottom-5 left-5 text-[8px] font-bold tracking-[.16em] text-white drop-shadow"
            >{temperature === "warm"
              ? "01 / LATE AFTERNOON"
              : "02 / BLUE HOUR"}</span
          >
        </div>
        <div class="self-center p-6 sm:p-10">
          <p
            class="mb-3 text-[9px] font-bold tracking-[.16em] text-neutral-500"
          >
            {temperature === "warm" ? "ENERGY / CLOSENESS" : "CALM / DISTANCE"}
          </p>
          <h3 class="text-3xl font-medium tracking-[-.05em]">
            {temperature === "warm"
              ? "Warm feels closer."
              : "Cool feels quieter."}
          </h3>
          <p class="mt-3 text-xs leading-7 text-neutral-600">
            {temperature === "warm"
              ? "Reds, oranges, and yellows often suggest heat, energy, urgency, or comfort. They can draw the eye forward."
              : "Blues, blue-greens, and violets often suggest calm, space, or focus. They can make a composition feel more distant."}
          </p>
          <div
            class="mt-5 divide-y divide-neutral-100 border-y border-neutral-200"
          >
            {#each temperature === "warm" ? warmColors : coolColors as item}
              <div
                class="grid grid-cols-[12px_65px_1fr] items-center gap-3 py-3 text-[10px]"
              >
                <span
                  class="h-2.5 w-2.5 rounded-full"
                  style={`background:${item.color}`}
                ></span><b>{item.name}</b><span class="text-neutral-500"
                  >{item.meaning}</span
                >
              </div>
            {/each}
          </div>
          <p class="mt-4 text-[9px] leading-5 text-neutral-500">
            Color meanings vary by culture, personal experience, and situation.
            Treat these as starting points, not universal facts.
          </p>
        </div>
      </div>
    </div>
  </section>

  <section
    id="harmony"
    class="scroll-mt-20 border-t border-neutral-200 py-16 md:py-20"
  >
    <div class="mx-auto max-w-7xl px-5 md:px-10">
      <div
        class="mb-10 flex flex-col justify-between gap-5 md:flex-row md:items-end"
      >
        <div>
          <p
            class="mb-3 text-[9px] font-bold tracking-[.18em] text-neutral-500"
          >
            04 / COLOR HARMONY
          </p>
          <h2 class="text-4xl font-medium tracking-[-.055em] sm:text-5xl">
            Colors in good company.
          </h2>
        </div>
        <p class="max-w-sm text-sm leading-7 text-neutral-600">
          A color scheme is a relationship between hues. Choose a rule, then see
          how it changes the palette.
        </p>
      </div>
      <div class="grid border border-neutral-200 md:grid-cols-[.72fr_1.28fr]">
        <div class="border-b border-neutral-200 p-5 md:border-b-0 md:border-r">
          <p
            class="mb-4 text-[8px] font-bold tracking-[.15em] text-neutral-500"
          >
            CHOOSE A RELATIONSHIP
          </p>
          {#each schemeOptions as option, i}
            <button
              class={`grid w-full grid-cols-[20px_1fr_12px] items-center gap-2 border px-2 py-4 text-left transition ${activeScheme === option.id ? "border-neutral-200 bg-neutral-100" : "border-transparent border-b-neutral-100 hover:bg-neutral-50"}`}
              aria-pressed={activeScheme === option.id}
              onclick={() => (activeScheme = option.id)}
            >
              <span class="self-start pt-0.5 text-[9px] text-neutral-400"
                >0{i + 1}</span
              ><span
                ><b class="block text-[11px] font-semibold">{option.title}</b
                ><small class="mt-1 block text-[9px] leading-5 text-neutral-500"
                  >{option.subtitle}</small
                ></span
              ><span class="text-xs text-neutral-400">↗</span>
            </button>
          {/each}
          <div class="mt-6 border-t border-neutral-200 px-2 pt-5">
            <label
              for="base-hue"
              class="mb-4 flex justify-between text-[8px] font-bold tracking-[.13em]"
              >BASE HUE <output>{Math.round(selectedHue)}°</output></label
            ><input
              id="base-hue"
              aria-label="Base hue for harmony scheme"
              type="range"
              min="0"
              max="359"
              bind:value={selectedHue}
              class="w-full accent-neutral-900"
            />
            <p class="mt-3 text-[9px] leading-5 text-neutral-500">
              Move the hue to generate a new scheme.
            </p>
          </div>
        </div>
        <div class="min-w-0 p-5 sm:p-7">
          <div
            class="flex justify-between text-[8px] font-bold tracking-[.15em] text-neutral-500"
          >
            <span
              >{schemeOptions
                .find((s) => s.id === activeScheme)
                ?.title.toUpperCase()}</span
            ><span>{palette.length} COLORS</span>
          </div>
          <div class="mt-5 flex h-52 gap-1.5 sm:h-60">
            {#each palette as color}
              <button
                class="group flex min-w-0 flex-1 flex-col items-start justify-between p-2.5 text-left text-white transition-all duration-200 hover:flex-[1.2] sm:p-3"
                style={`background:${color.color};text-shadow:0 1px 3px #0004`}
                aria-label={`Copy ${color.label} ${color.hex}`}
                onclick={() => copyHex(color.hex)}
                ><span
                  class="text-[9px] font-semibold [writing-mode:vertical-rl] rotate-180"
                  >{color.label}</span
                ><span class="text-[9px] font-bold">{color.hex}</span></button
              >
            {/each}
          </div>
          <div
            class="mt-3 flex justify-between text-[8px] tracking-[.12em] text-neutral-400"
          >
            <span>01 — HUE RELATIONSHIPS</span><span>GENERATED LIVE ↗</span>
          </div>
          <div
            class="mt-6 grid items-center gap-5 border-t border-neutral-200 pt-5 sm:grid-cols-[145px_1fr]"
          >
            <div class="grid h-32 place-items-center bg-neutral-50">
              <svg
                viewBox="0 0 160 140"
                role="img"
                aria-label={`${schemeOptions.find((s) => s.id === activeScheme)?.title} hue relationship diagram`}
                class="w-36"
              >
                <circle
                  cx="80"
                  cy="70"
                  r="52"
                  fill="none"
                  stroke="#e5e5e5"
                /><circle cx="80" cy="70" r="38" fill="none" stroke="#e5e5e5" />
                {#each palette as color}
                  <line
                    x1="80"
                    y1="70"
                    x2={80 + 48 * Math.cos(((color.hue - 90) * Math.PI) / 180)}
                    y2={70 + 48 * Math.sin(((color.hue - 90) * Math.PI) / 180)}
                    stroke={hsl(color.hue, saturation, 50)}
                    stroke-width="2"
                  />
                  <circle
                    cx={80 + 48 * Math.cos(((color.hue - 90) * Math.PI) / 180)}
                    cy={70 + 48 * Math.sin(((color.hue - 90) * Math.PI) / 180)}
                    r="7"
                    fill={hsl(color.hue, saturation, 50)}
                    stroke="white"
                    stroke-width="2"
                  />
                {/each}
              </svg>
            </div>
            <div>
              <h3 class="text-base font-semibold tracking-tight">
                {schemeOptions.find((s) => s.id === activeScheme)?.title}
              </h3>
              <p class="mt-2 text-[10px] leading-6 text-neutral-600">
                {activeScheme === "complementary"
                  ? "Pick hues opposite each other on the wheel. Their difference creates strong contrast — useful for emphasis, focal points, and calls to action."
                  : activeScheme === "analogous"
                    ? "Pick neighboring hues. Their similar undertones feel connected and are easy to blend into a calm, cohesive palette."
                    : activeScheme === "split"
                      ? "Pair one hue with the two hues beside its complement. You keep contrast, but the result can feel less direct than a simple complementary pair."
                      : "Choose three hues spaced roughly 120° apart. Let one color dominate and use the others as accents for variety and balance."}
              </p>
            </div>
          </div>
          {#if copied}<p
              aria-live="polite"
              class="mt-3 text-[10px] text-neutral-500"
            >
              {copied.startsWith("#")
                ? `${copied} copied to clipboard.`
                : copied}
            </p>{/if}
        </div>
      </div>
      <div
        class="mt-6 flex flex-wrap items-start gap-4 border-t border-neutral-200 pt-5"
      >
        <span
          class="shrink-0 text-[8px] font-bold tracking-[.14em] text-neutral-400"
          >DESIGN NOTE</span
        >
        <p class="text-xs leading-6 text-neutral-600">
          Harmony is a starting point, not a formula. Use contrast, proportion,
          and context to decide which color leads.
        </p>
      </div>
    </div>
  </section>
</main>
