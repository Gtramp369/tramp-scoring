<script lang="ts">
    import * as Select from "$lib/components/ui/select";
    import * as Card from "$lib/components/ui/card";
    import * as Button from "$lib/components/ui/button";
    import { Checkbox } from "$lib/components/ui/checkbox/index.js";
    import { Input } from "$lib/components/ui/input/index.js";
    import {Sun, Moon, GithubLogo} from "svelte-radix";
    import { toggleMode } from "mode-watcher";
    
    let selected = { value: 0, label: "" };
    
    const comboLength = [
      { value: 1, label: "1 Trick" },
      { value: 2, label: "2 Tricks" },
      { value: 3, label: "3 Tricks" },
      { value: 4, label: "4 Tricks" },
      { value: 5, label: "5 Tricks" },
    ];

    const transitionOptions = [
      { value: 0.15, label: "back feet" },
      { value: 0.3, label: "front feet" },
      { value: 0.1, label: "back cody" },
      { value: 0.3, label: "front cody" },
      { value: 0, label: "back kaboom" },
      { value: 0.2, label: "front kaboom" },
      { value: 0, label: "back pullover" },
      { value: 0.4, label: "front pullover" },
      { value: 0.1, label: "front ballout" },
      { value: 0.2, label: "back ballout" },
      { value: 0.0, label: "front benny" },
      { value: 0.35, label: "back benny" },
      { value: 0.25, label: "front zach" },
      { value: 0.4, label: "back zach" },
    ];
    
    // Add a "None" option with value 0 for transitions
    const transitionOptionsWithNone = [
      { value: 0, label: "None" },
      ...transitionOptions
    ];
    
    interface Trick {
      flips: number | undefined;
      twists: number | undefined;
      transition: { value: number; label: string };
      landed: boolean;
    }
    
    let tricks: Trick[] = [];
    
    $: if (selected.value > 0) {
      // Update the tricks array to match the selected number of tricks
      tricks = Array.from({ length: selected.value }, (_, i) => ({
        flips: undefined,
        twists: undefined,
        transition: { value: 0, label: "None" },
        landed: false, // Add 'landed' property to each trick
      }));
    }
    
    let totalScore = 0; // Track the total score
    
    function calculateScore() {
        const calculateScore = (flips: number | undefined, twists: number | undefined) => {
            if (flips === undefined) {
                flips = 0
            }
            if (twists === undefined) {
                twists = 0
            }
            return 0.04 * (3 * Math.pow(flips, 1.5) + 2 * Math.pow(twists + 0.2 * flips, 1.5));
        };

    
      totalScore = 0; // Reset total score before calculating
    
      tricks.forEach((trick, index) => {
        const flips = trick.flips;
        const twists = trick.twists;
        const transition = transitionOptionsWithNone.find(t => t.label === trick.transition.label); // Correctly find transition
        let score = calculateScore(flips, twists);
    
        // If it's the last trick, add the landed bonus
        if (index === tricks.length - 1) {
          score += trick.landed ? 0.1 : 0;
        } else if (transition) {
          score += transition.value; // Correctly apply the transition value
        }
    
        totalScore += score;
      });
    }
    
    // Append unique identifiers to each value
    const uniqueTransitionOptions = transitionOptions.map((transition, index) => ({
      ...transition,
      uniqueValue: `${transition.value}-${index}`, // Append index to make each value unique
    }));
    
    // Track if a select is open
    let isSelectOpen = false;
    
    // Update scroll behavior based on select state (only runs on client-side)
    $: {
      if (typeof window !== "undefined") {
        // Ensure this code runs only on the client
        if (isSelectOpen) {
          document.body.style.overflow = "auto"; // Enable body scroll when select is open
        } else {
          document.body.style.overflow = ""; // Reset body scroll when select is closed
        }
      }
    }
  </script>
  
  <div class="mx-5">
    <h1 class="text-center text-5xl mt-20 italic">World champs scoring system</h1>
    <div class="mx-auto flex justify-center mt-10">
      <Select.Root bind:selected={selected} on:open={e => isSelectOpen = true} on:close={e => isSelectOpen = false}>
        <div class="flex flex-col justify-center">
          <Select.Label class="text-center text-lg">Number of tricks in the combo (1-5)</Select.Label>
          <Select.Trigger>
            <Select.Value placeholder="Select a combo length" />
          </Select.Trigger>
          <Select.Content class="max-h-60 overflow-y-auto"> <!-- Added max-height and overflow handling -->
            <Select.Group>
              {#each comboLength as length}
                <Select.Item value={length.value} label={length.label}>{length.label}</Select.Item>
              {/each}
            </Select.Group>
          </Select.Content>
        </div>
        <Select.Input name="comboLength" />
      </Select.Root>
    </div>
  
    <p class="text-center my-5">Selected combo length: {selected.value}</p>
  
    <!-- Generate cards dynamically based on the selected number of tricks -->
    <div class="flex flex-wrap justify-center gap-4">
      {#each tricks as trick, index}
        <Card.Root class="p-5 shadow-md w-full sm:w-[300px] lg:w-[450px]">
          <Card.Header>
            <Card.Title>Trick {index + 1}</Card.Title>
          </Card.Header>
          <Card.Content class="flex flex-col gap-6">
            <div>
                <p class="mb-1">Number of flips</p>
                <Input type="number" placeholder="Number of flips" bind:value={trick.flips} />
            </div>
            <div>
                <p class="mb-1">Number of twists</p>
                <Input type="number" placeholder="Number of twists" bind:value={trick.twists} />
            </div>
  
            {#if index === tricks.length - 1}
              <!-- Last trick, show checkbox instead of transition input -->
              <div>
                <label class="flex items-center">
                  <Checkbox bind:checked={trick.landed} class="mr-2" />
                  Landed
                </label>
              </div>
            {:else}
              <div>
                <Select.Root bind:selected={trick.transition} preventScroll={false}>
                  <p class="mb-1">Transition</p>
                  <Select.Trigger>
                    <Select.Value placeholder="Select a transition" />
                  </Select.Trigger>
                  <Select.Content>
                    <Select.Group>
                      {#each uniqueTransitionOptions as transition}
                        <Select.Item value={transition.uniqueValue} label={transition.label}>{transition.label}</Select.Item>
                      {/each}
                    </Select.Group>
                  </Select.Content>
                </Select.Root>
              </div>
            {/if}
          </Card.Content>
        </Card.Root>
      {/each}
    </div>
  
    <div class="flex justify-center my-5">
      <Button.Root on:click={calculateScore} class="px-5 bg-sky-600 hover:bg-sky-400 transition duration-200">Calculate Score</Button.Root>
    </div>
    <p class="text-center text-xl mt-5 mb-10">Total Score: {totalScore}</p>
    <p class="text-center text-xl italic">Current formula (the Hugo formula):</p>
    <p class="text-center mb-10">Score = 0.04 × (3(flips^1.5) + 2((twists+0.2×flips)^1.5)) + transition bonus</p>
  
  </div>
  <div class="mx-auto flex justify-center gap-10 mb-20">

    <Button.Root on:click={toggleMode} variant="outline" size="icon" class="rounded-full">
      <Sun
        class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
      />
      <Moon
        class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
      />
      <span class="sr-only">Toggle theme</span>
    </Button.Root>
    <a href="https://github.com/Gtramp369/tramp-scoring" target="_blank">
     <div class="flex-col hover:text-purple-500 transition duration-200">
         <GithubLogo class="mx-auto" />
         <p>Code</p>
  </div>
 </a>
</div>
