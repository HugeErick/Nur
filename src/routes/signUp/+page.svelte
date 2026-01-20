<script>
// signup page
import * as z from "zod";
import { goto } from "$app/navigation";
import { Button } from "$lib/components/ui/button/index.js";
import { Label } from "$lib/components/ui/label/index.js";
import { Input } from "$lib/components/ui/input/index.js";
import * as Card from "$lib/components/ui/card/index.js";
import { Loader2 } from "@lucide/svelte";

let username = "";
let email = "";
let password = "";
let confirmPassword = "";
let message = "";
let isLoading = false; 

let errors = {};

const signUpSchema = z.object({
  username: z.string().min(3, "Username too short").max(50),
  email: z.string().email("Invalid email address"),
  password: z.string().min(4, "Password must be +4 characters"),
  confirmPassword: z.string()
})
.strict() // dont allow extra fields from request  
.refine((data) => data.password === data.confirmPassword, {
  message: "Password don't match",
  path: ["confirmPassword"], // puts err in confirmPassword field
})

async function handleSignUp() {
  errors = {} //clear prev errs
  message = "";

  const result = signUpSchema.safeParse({ username, email, password, confirmPassword });

  if (!result.success) {
    errors = result.error.flatten().fieldErrors;
    return;
  }

  isLoading = true; 
  try {
    const response = await fetch("https://nurichvsdiewelt.work/nur/nur-register", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ username, email, password })
    });

    const resultData = await response.json();
    if (response.ok) {
      message = "Success! Account created.";
      goto("/");
    } else {
      message = resultData.error || "Registration failed";
    }
  } catch (err) {
    message = "Connection error to server";
  } finally {
    isLoading = false; 
  }
}
</script>

<section class="min-h-screen w-full flex items-center justify-center p-4 py-12">
  <Card.Root class="w-full max-w-sm">
    <Card.Header>
      <Card.Title class="text-3xl text-(--customGold)">Sign Up</Card.Title>    
    </Card.Header>

    <form on:submit|preventDefault={handleSignUp}>
      <Card.Content class="my-2">
        <div class="flex flex-col gap-4">
          <div class="grid gap-2">
            <Label for="nur-username">Create Username</Label>
            <Input id="nur-username" bind:value={username} placeholder="Magnus Carlsen" required />
            {#if errors.username}
              <p class="text-xs text-red-500">{errors.username[0]}</p>
            {/if}
          </div>

          <div class="grid gap-2">
            <Label for="nur-mail">Email</Label>
            <Input id="nur-mail" type="email" bind:value={email} placeholder="abc@example.com" required />

            <Card.Description>
              We'll use this to contact you.
            </Card.Description>
          </div>

          <div class="grid gap-2">
            <Label for="nur-password">Password</Label>
            <Input id="nur-password" bind:value={password} type="password" required />
          </div>
          {#if errors.Password}
            <p class="text-xs text-red-500">{errors.Password[0]}</p>
          {/if}

          <div class="grid gap-2">
            <Label for="confirm-password">Confirm Password</Label>
            <Input id="confirm-password" bind:value={confirmPassword} type="password" required />
            {#if errors.confirmPassword}
              <p class="text-xs text-red-500">{errors.confirmPassword[0]}</p>
            {/if}
          </div>
        </div>
      </Card.Content>

      <Card.Footer class="mt-4">
        <div class="flex flex-col w-full gap-2">
          <Card.Action class="w-full">
            <Button type="submit" class="w-full" disabled={isLoading}>
              {#if isLoading}
                <Loader2 class="mr-2 h-4 w-4 animate-spin" />
                Please wait
              {:else}
                Sign up
              {/if}
            </Button>
          </Card.Action>

          {#if message}
            <p class="text-sm text-center {message.toLowerCase().includes('success') ? 'text-green-500' : 'text-red-500'}">
              {message}
            </p>
          {/if}
        </div>
      </Card.Footer>
    </form>
  </Card.Root>
</section>
