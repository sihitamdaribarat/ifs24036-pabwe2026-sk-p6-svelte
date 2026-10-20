<script>
  import { onMount } from "svelte";
  import { navigate } from "../../../router.js";
  import AddModal from "../modals/AddModal.svelte";
  import ChangeModal from "../modals/ChangeModal.svelte";
  import { linkTreesStore } from "../states/linkTreesStore";
  import { usersStore } from "../../users/states/usersStore";
  import { formatDate, showConfirmDialog } from "../../../helpers/toolsHelper";
  import {
    Plus,
    Link,
    Eye,
    Pencil,
    Trash2,
    Funnel,
    Search,
    LoaderCircle,
    Globe,
    Lock,
    EyeOff,
  } from "lucide-svelte";

  let loadingLinkTrees = false;
  let filter = ""; // "" for Semua, "1" for Milik Saya
  let searchQuery = "";
  let showAddModal = false;
  let showChangeModal = false;
  let selectedLinkTreeId = null;

  $: profile = $usersStore.profile;
  $: linkTrees = $linkTreesStore.linkTrees || [];

  onMount(() => {
    loadLinkTrees();
  });

  function loadLinkTrees() {
    loadingLinkTrees = true;
    Promise.resolve(linkTreesStore.asyncSetLinkTrees(filter)).finally(() => {
      loadingLinkTrees = false;
    });
  }

  let prevFilter = filter;
  $: if (filter !== prevFilter) {
    prevFilter = filter;
    loadLinkTrees();
  }

  $: if ($linkTreesStore.isLinkTreeDeleted) {
    linkTreesStore.setIsLinkTreeDeleted(false);
    loadLinkTrees();
  }

  $: if ($linkTreesStore.isLinkTreeDeletedAll) {
    linkTreesStore.setIsLinkTreeDeletedAll(false);
    loadLinkTrees();
  }

  function handleEditLinkTree(id) {
    selectedLinkTreeId = id;
    showChangeModal = true;
  }

  async function handleDeleteLinkTree(id) {
    const result = await showConfirmDialog("Apakah Anda yakin ingin menghapus Link Tree ini?");
    if (result.isConfirmed) {
      linkTreesStore.asyncSetIsLinkTreeDelete(id);
    }
  }

  async function handleDeleteAll() {
    const result = await showConfirmDialog("Apakah Anda yakin ingin menghapus SEMUA Link Tree milik Anda?");
    if (result.isConfirmed) {
      linkTreesStore.asyncSetIsLinkTreeDeleteAll();
    }
  }

  $: filteredLinkTrees = linkTrees.filter((item) => {
    if (!searchQuery.trim()) return true;
    const q = searchQuery.toLowerCase();
    const title = item.title ? item.title.toLowerCase() : "";
    const description = item.description ? item.description.toLowerCase() : "";
    return title.includes(q) || description.includes(q);
  });

  $: totalCount = linkTrees.length;
  $: publicCount = linkTrees.filter((l) => l.type === "public").length;
  $: myCount = profile ? linkTrees.filter((l) => l.user_id === profile.id).length : 0;
</script>

{#if profile}
  <div class="space-y-8 animate-in fade-in duration-300">
    <!-- Header Banner -->
    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
      <div>
        <h1 class="text-2xl sm:text-3xl font-extrabold text-slate-900 tracking-tight">
          Daftar Link Tree
        </h1>
        <p class="text-sm text-slate-600 mt-1">
          Kelola dan bagikan seluruh kumpulan tautan penting Anda dalam satu tempat.
        </p>
      </div>
      <div class="flex items-center gap-2.5 self-start sm:self-auto">
        <button
          type="button"
          data-testid="delete-all-btn"
          on:click={handleDeleteAll}
          class="inline-flex items-center justify-center gap-2 px-3.5 py-2.5 rounded-xl font-bold text-sm text-red-800 bg-red-50 hover:bg-red-100 border border-red-200 transition-all"
          title="Hapus Semua Link Tree Milik Saya"
        >
          <Trash2 size={16} />
          <span>Hapus Semua</span>
        </button>
        <button
          type="button"
          data-testid="add-linktree-btn"
          on:click={() => (showAddModal = true)}
          class="inline-flex items-center justify-center gap-2 px-4 py-2.5 rounded-xl font-semibold text-sm text-white bg-indigo-600 hover:bg-indigo-700 active:bg-indigo-800 shadow-md shadow-indigo-600/25 transition-all"
        >
          <Plus size={18} strokeWidth={2.5} />
          <span>Tambah Link Tree</span>
        </button>
      </div>
    </div>

    <!-- Stats Cards -->
    <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
      <div class="bg-white rounded-2xl p-5 border border-slate-200/80 shadow-xs flex items-center justify-between">
        <div>
          <p class="text-xs font-semibold uppercase tracking-wider text-slate-700">
            Total Link Tree
          </p>
          <h3 class="text-3xl font-black text-slate-800 mt-1">{totalCount}</h3>
        </div>
        <div class="w-12 h-12 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center">
          <Link size={26} strokeWidth={2} />
        </div>
      </div>

      <div class="bg-white rounded-2xl p-5 border border-slate-200/80 shadow-xs flex items-center justify-between">
        <div>
          <p class="text-xs font-semibold uppercase tracking-wider text-slate-700">
            Link Tree Publik
          </p>
          <h3 class="text-3xl font-black text-emerald-700 mt-1">
            {publicCount}
          </h3>
        </div>
        <div class="w-12 h-12 rounded-2xl bg-emerald-50 text-emerald-700 flex items-center justify-center">
          <Globe size={26} strokeWidth={2} />
        </div>
      </div>

      <div class="bg-white rounded-2xl p-5 border border-slate-200/80 shadow-xs flex items-center justify-between">
        <div>
          <p class="text-xs font-semibold uppercase tracking-wider text-slate-700">
            Milik Saya
          </p>
          <h3 class="text-3xl font-black text-amber-700 mt-1">{myCount}</h3>
        </div>
        <div class="w-12 h-12 rounded-2xl bg-amber-50 text-amber-700 flex items-center justify-center">
          <Lock size={26} strokeWidth={2} />
        </div>
      </div>
    </div>

    <!-- Table & Controls Section -->
    <div class="bg-white rounded-3xl border border-slate-200/80 shadow-xs overflow-hidden">
      <!-- Filter bar -->
      <div class="p-4 sm:p-5 border-b border-slate-100 flex flex-col md:flex-row md:items-center justify-between gap-4">
        <div class="relative flex-1 max-w-md">
          <Search
            size={18}
            class="absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-600"
          />
          <input
            type="text"
            data-testid="search-linktree-input"
            bind:value={searchQuery}
            placeholder="Cari judul atau rincian link tree..."
            class="w-full pl-10 pr-4 py-2 text-sm rounded-xl border border-slate-200 bg-white placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 transition-all"
          />
        </div>

        <div class="flex items-center gap-2.5">
          <span class="text-xs font-semibold text-slate-700 uppercase tracking-wide flex items-center gap-1.5">
            <Funnel size={16} /> Filter:
          </span>
          <div class="inline-flex rounded-xl bg-slate-100 p-1 text-xs font-semibold text-slate-700">
            <button
              type="button"
              data-testid="filter-all-btn"
              on:click={() => (filter = "")}
              class="px-3 py-1.5 rounded-lg transition-all {filter === '' ? 'bg-white text-slate-900 shadow-xs' : 'hover:text-slate-900'}"
            >
              Semua
            </button>
            <button
              type="button"
              data-testid="filter-mine-btn"
              on:click={() => (filter = "1")}
              class="px-3 py-1.5 rounded-lg transition-all {filter === '1' ? 'bg-white text-indigo-700 shadow-xs' : 'hover:text-slate-900'}"
            >
              Milik Saya
            </button>
          </div>
        </div>
      </div>

      <!-- Responsive Table -->
      <div class="overflow-x-auto">
        <table class="w-full text-left text-sm text-slate-700">
          <thead class="bg-slate-50/80 text-xs uppercase tracking-wider font-semibold text-slate-700 border-b border-slate-100">
            <tr>
              <th class="px-5 py-3.5 text-center w-16">ID</th>
              <th class="px-5 py-3.5">Judul & Cover</th>
              <th class="px-5 py-3.5">Penulis</th>
              <th class="px-5 py-3.5">Visibilitas</th>
              <th class="px-5 py-3.5 text-center">Tautan</th>
              <th class="px-5 py-3.5 hidden md:table-cell">Dibuat</th>
              <th class="px-5 py-3.5 text-right">Aksi</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100">
            {#if loadingLinkTrees && filteredLinkTrees.length === 0}
              <tr>
                <td colspan="7" class="px-6 py-12 text-center text-slate-600">
                  <LoaderCircle size={36} class="mx-auto text-indigo-600 animate-spin mb-2" />
                  <p class="font-medium text-slate-700">Memuat daftar link tree...</p>
                </td>
              </tr>
            {:else if filteredLinkTrees.length === 0}
              <tr>
                <td colspan="7" class="px-6 py-12 text-center text-slate-600">
                  <Link size={40} class="mx-auto text-slate-600 mb-2" />
                  <p class="font-medium text-slate-700">Belum ada data link tree yang cocok.</p>
                </td>
              </tr>
            {:else}
              {#each filteredLinkTrees as item (item.id)}
                <tr
                  data-testid={`linktree-row-${item.id}`}
                  class="hover:bg-slate-50/70 transition-colors group"
                >
                  <td class="px-5 py-4 text-center font-mono text-xs font-bold text-slate-800">
                    {"#" + item.id}
                  </td>
                  <td class="px-5 py-4">
                    <div class="flex items-center gap-3">
                      {#if item.cover}
                        <img
                          src={item.cover}
                          alt={item.title}
                          class="w-12 h-12 rounded-xl object-cover border border-slate-200 shrink-0"
                        />
                      {:else}
                        <div class="w-12 h-12 rounded-xl bg-slate-100 text-slate-600 flex items-center justify-center shrink-0 border border-slate-200">
                          <Link size={20} />
                        </div>
                      {/if}
                      <div>
                        <p class="font-semibold text-slate-800 leading-snug">
                          {item.title}
                        </p>
                        {#if item.description}
                          <p class="text-xs text-slate-600 line-clamp-1 mt-0.5">
                            {item.description}
                          </p>
                        {/if}
                      </div>
                    </div>
                  </td>
                  <td class="px-5 py-4 text-xs text-slate-600">
                    <div class="flex items-center gap-2">
                      {#if item.author?.photo}
                        <img
                          src={item.author.photo}
                          alt={item.author.name}
                          class="w-6 h-6 rounded-full object-cover border border-slate-200"
                        />
                      {/if}
                      <span class="font-medium">{item.author?.name || "Anonim"}</span>
                    </div>
                  </td>
                  <td class="px-5 py-4">
                    {#if item.type === "public"}
                      <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-xs font-semibold bg-emerald-50 text-emerald-700 border border-emerald-200/60">
                        <Globe size={12} />
                        Publik
                      </span>
                    {:else if item.type === "unlisted"}
                      <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-xs font-semibold bg-amber-50 text-amber-700 border border-amber-200/60">
                        <EyeOff size={12} />
                        Unlisted
                      </span>
                    {:else}
                      <span class="inline-flex items-center gap-1 px-2.5 py-1 rounded-full text-xs font-semibold bg-slate-100 text-slate-700 border border-slate-200">
                        <Lock size={12} />
                        Pribadi
                      </span>
                    {/if}
                  </td>
                  <td class="px-5 py-4 text-center">
                    <span class="inline-flex items-center justify-center px-2 py-0.5 rounded-md text-xs font-semibold bg-indigo-50 text-indigo-700">
                      {item.total_links ?? 0}
                    </span>
                  </td>
                  <td class="px-5 py-4 hidden md:table-cell text-xs text-slate-600">
                    {formatDate(item.created_at)}
                  </td>
                  <td class="px-5 py-4 text-right">
                    <div class="inline-flex items-center gap-1.5">
                      <button
                        type="button"
                        data-testid={`view-linktree-${item.id}`}
                        on:click={() => navigate(`/link-trees/${item.id}`)}
                        class="p-1.5 text-slate-600 hover:text-indigo-600 hover:bg-indigo-50 rounded-lg transition-colors"
                        title="Lihat Detail"
                      >
                        <Eye size={18} />
                      </button>
                      <button
                        type="button"
                        data-testid={`edit-linktree-${item.id}`}
                        on:click={() => handleEditLinkTree(item.id)}
                        class="p-1.5 text-slate-600 hover:text-amber-600 hover:bg-amber-50 rounded-lg transition-colors"
                        title="Ubah Link Tree"
                      >
                        <Pencil size={18} />
                      </button>
                      <button
                        type="button"
                        data-testid={`delete-linktree-${item.id}`}
                        on:click={() => handleDeleteLinkTree(item.id)}
                        class="p-1.5 text-slate-600 hover:text-red-600 hover:bg-red-50 rounded-lg transition-colors"
                        title="Hapus Link Tree"
                      >
                        <Trash2 size={18} />
                      </button>
                    </div>
                  </td>
                </tr>
              {/each}
            {/if}
          </tbody>
        </table>
      </div>
    </div>

    <!-- Modals -->
    <AddModal show={showAddModal} on:close={() => (showAddModal = false)} />
    <ChangeModal
      show={showChangeModal}
      linkTreeId={selectedLinkTreeId}
      on:close={() => (showChangeModal = false)}
    />
  </div>
{/if}
