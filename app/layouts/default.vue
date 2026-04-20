<template>
  <div class="flex flex-col min-h-screen">
    <TheHeader />
    <div class="drawer flex-1">
      <input id="main-drawer" type="checkbox" class="drawer-toggle" />
      <div class="drawer-content flex flex-col lg:ml-20">
        <main class="flex-grow">
          <slot />
        </main>
        <TheFooter />
      </div>
      <Drawer />
    </div>
  </div>
</template>
