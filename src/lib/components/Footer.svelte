<script lang="ts">
    import {Sun, Moon, GithubLogo} from "svelte-radix";
    import { toggleMode } from "mode-watcher";
    import { Button } from "$lib/components/ui/button/index.js";
  </script>
   <div class="mx-auto flex justify-center gap-10">

       <Button on:click={toggleMode} variant="outline" size="icon" class="rounded-full">
         <Sun
           class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
         />
         <Moon
           class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
         />
         <span class="sr-only">Toggle theme</span>
       </Button>
       <a href="https://github.com/Gtramp369/tramp-scoring" target="_blank">
        <div class="flex-col hover:text-purple-500 transition duration-200">
            <GithubLogo class="mx-auto" />
            <p>Code</p>
     </div>
    </a>
   </div>
  
  