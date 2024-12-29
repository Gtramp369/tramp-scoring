<script lang="ts">
    import * as Select from "$lib/components/ui/select";
    import * as Card from "$lib/components/ui/card";
    import * as Button from "$lib/components/ui/button";
    import { Checkbox } from "$lib/components/ui/checkbox/index.js";
    import { Input } from "$lib/components/ui/input/index.js";
    import {Sun, Moon, GithubLogo} from "svelte-radix";
    import { toggleMode } from "mode-watcher";
    
    let selected = { value: 1, label: "1 Trick" };
    
    const comboLength = [
      { value: 1, label: "1 Trick" },
      { value: 2, label: "2 Tricks" },
      { value: 3, label: "3 Tricks" },
      { value: 4, label: "4 Tricks" },
      { value: 5, label: "5 Tricks" },
    ];

    const transitionOptions = [
      { value: 0.15, label: "back feet + 0.15" },
      { value: 0.3, label: "front feet + 0.3" },
      { value: 0.1, label: "back cody + 0.1" },
      { value: 0.3, label: "front cody + 0.3" },
      { value: 0.0, label: "back kaboom + 0.0" },
      { value: 0.2, label: "front kaboom + 0.2" },
      { value: 0.0, label: "back pullover + 0" },
      { value: 0.4, label: "front pullover + 0.4" },
      { value: 0.1, label: "front ballout + 0.1" },
      { value: 0.2, label: "back ballout + 0.2" },
      { value: 0.0, label: "front benny + 0.0" },
      { value: 0.35, label: "back benny + 0.35" },
      { value: 0.25, label: "front zach + 0.25" },
      { value: 0.4, label: "back zach + 0.4" },
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

        const calculateScoreForTrick = (flips: number | undefined, twists: number | undefined) => {
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
        const transition = transitionOptions.find(t => t.label === trick.transition.label); // Find transition
        let score = calculateScoreForTrick(flips, twists);
    
        // If it's the last trick, add the landed bonus
        if (index === tricks.length - 1) {
          score += trick.landed ? 0.1 : 0;
        } else if (transition) {
          score += transition.value; // Apply the transition value
        }
    
        totalScore += score;
      });
    }
    
    // Append unique identifiers to each value
    const uniqueTransitionOptions = transitionOptions.map((transition, index) => ({
      ...transition,
      uniqueValue: `${transition.value}-${index}`, // Append index to make each value unique
    }));
    
  </script>
  
  <div class="mx-5">
    <h1 class="text-center text-5xl mt-10 italic">Difficulty calculator</h1>
    <div class="mx-auto flex justify-center my-10">
      <Select.Root bind:selected={selected}>
        <div class="flex flex-col justify-center">
          <Select.Label class="text-center text-lg">Number of tricks in the combo (1-5)</Select.Label>
          <Select.Trigger>
            <Select.Value placeholder="Select a combo length" />
          </Select.Trigger>
          <Select.Content>
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
  
  
    <!-- Generate cards dynamically based on the selected number of tricks -->
    <div class="flex flex-wrap justify-center gap-5 max-w-screen-2xl mx-auto">
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
    <p class="text-center text-xl my-5">Total Score: {totalScore}</p>
    <p class="text-center text-xl italic">Current formula (Hugo's formula):</p>
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
