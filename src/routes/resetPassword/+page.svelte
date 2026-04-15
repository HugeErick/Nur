<script lang="ts">
  import * as z from "zod";
  import { page } from "$app/stores";
  import { Button } from "$lib/components/ui/button/index.js";
  import { Label } from "$lib/components/ui/label/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import * as Card from "$lib/components/ui/card/index.js";
  import { Loader2 } from "@lucide/svelte";

  let newPassword = "";
  let confirmPassword = "";
  let message = "";
  let isLoading = false;

  const token = $page.url.searchParams.get("token");

  const schema = z.object({
    newPassword: z.string().min(6, "Password must be at least 6 characters"),
    confirmPassword: z.string()
  }).refine(data => data.newPassword === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"]
  });

  async function handleReset() {
    message = "";

    if (!token) {
      message = "Invalid or missing reset token";
      return;
    }

    const result = schema.safeParse({ newPassword, confirmPassword });
    if (!result.success) {
      message = result.error.issues[0].message;
      return;
    }

    isLoading = true;
    try {
      const res = await fetch("https://nurichvsdiewelt.work/nur/nur-reset-password", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ token, newPassword }),
      });
      const data = await res.json();
      message = res.ok ? "Password updated! You can now log in." : data.error || "Reset failed";
    } catch {
      message = "Could not connect to server";
    } finally {
      isLoading = false;
    }
  }
</script>

<section class="w-screen h-screen">
  <Card.Root class="absolute p-8 w-4xl max-w-sm top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">
    <Card.Header>
      <Card.Title class="text-3xl text-(--customGold)">Reset Password</Card.Title>
    </Card.Header>

    <form on:submit|preventDefault={handleReset}>
      <Card.Content>
        <div class="flex flex-col gap-6">
          <div class="grid gap-2">
            <Label for="new-password">New Password</Label>
            <Input id="new-password" bind:value={newPassword} type="password" required />
          </div>
          <div class="grid gap-2">
            <Label for="confirm-password">Confirm Password</Label>
            <Input id="confirm-password" bind:value={confirmPassword} type="password" required />
          </div>
        </div>
      </Card.Content>
      <Card.Footer class="flex flex-col mt-2 w-full">
        <Card.Action class="w-full">
          <Button type="submit" class="w-full" disabled={isLoading}>
            {#if isLoading}
              <Loader2 class="mr-2 h-4 w-4 animate-spin" />
              Please wait
            {:else}
              Reset Password
            {/if}
          </Button>
        </Card.Action>
        {#if message}
          <p class="text-sm mt-2
            {message.includes('now log in') ? 'text-green-500' : 'text-red-500'}">
            {message}
          </p>
        {/if}
      </Card.Footer>
    </form>
  </Card.Root>
</section>
