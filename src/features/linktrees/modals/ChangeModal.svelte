<script>
  import { createEventDispatcher } from "svelte";
  import { PenLine, X, LoaderCircle } from "lucide-svelte";
  import { linkTreesStore } from "../states/linkTreesStore";
  import linkTreeApi from "../api/linkTreeApi";
  import { showErrorDialog } from "../../../helpers/toolsHelper";

  export let show = false;
  export let linkTreeId = null;
  export let onClose = undefined;

  const dispatch = createEventDispatcher();

  let loading = false;
  let fetchingLinkTree = false;
  let title = "";
  let type = "public";
  let description = "";

  function handleClose() {
    dispatch("close");
    if (onClose) onClose();
  }

  $: {
    if (show) {
      document.body.style.overflow = "hidden";
    } else {
      document.body.style.overflow = "auto";
    }
  }

  $: if (show && linkTreeId) {
    loadLinkTreeData(linkTreeId);
  }

  async function loadLinkTreeData(id) {
    fetchingLinkTree = true;
    try {
      const data = await linkTreeApi.getLinkTreeById(id);
      if (data) {
        title = data.title || "";
        type = data.type || "public";
        description = data.description || "";
      }
    } catch {
      showErrorDialog("Gagal mengambil data link tree");
      handleClose();
    } finally {
      fetchingLinkTree = false;
    }
  }

  $: if ($linkTreesStore.isLinkTreeChange) {
    loading = false;
    linkTreesStore.setIsLinkTreeChange(false);
    if ($linkTreesStore.isLinkTreeChanged) {
      linkTreesStore.setIsLinkTreeChanged(false);
      linkTreesStore.asyncSetLinkTrees();
      linkTreesStore.asyncSetLinkTree(linkTreeId);
      handleClose();
    }
  }

  function handleSave() {
    if (!title.trim()) {
      showErrorDialog("Judul tidak boleh kosong");
      return;
    }

    if (!description.trim()) {
      showErrorDialog("Deskripsi tidak boleh kosong");
      return;
    }

    loading = true;
    linkTreesStore.asyncSetIsLinkTreeChange(
      linkTreeId,
      title.trim(),
      type,
      description.trim()
    );
  }
</script>

{#if show}
  <div
    data-testid="edit-linktree-modal"
    class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/50 backdrop-blur-xs animate-in fade-in duration-200"
  >
    <div
      class="w-full max-w-xl bg-white rounded-3xl shadow-2xl border border-slate-100 overflow-hidden flex flex-col max-h-[90vh]"
    >
      <!-- Modal Header -->
      <div class="flex items-center justify-between px-6 py-4 border-b border-slate-200 bg-slate-50/80 shrink-0">
        <div class="flex items-center gap-2.5">
          <div class="w-8 h-8 rounded-lg bg-amber-100 text-amber-700 flex items-center justify-center">
            <PenLine size={18} strokeWidth={2.5} />
          </div>
          <div>
            <h3 class="text-base font-bold text-slate-500">Ubah Data Link Tree</h3>
            <p class="text-xs text-slate-500">Perbarui judul, tipe visibilitas, dan detail deskripsi</p>
          </div>
        </div>
        <button
          type="button"
          data-testid="close-edit-modal-btn"
          on:click={handleClose}
          class="p-2 rounded-xl text-slate-500 hover:text-slate-500 hover:bg-slate-200/60 transition-colors"
        >
          <X size={20} />
        </button>
      </div>

      <!-- Modal Form Body -->
      <form on:submit|preventDefault={handleSave} class="flex-1 flex flex-col min-h-0 bg-white">
        <div class="p-6 md:p-8 space-y-4 overflow-y-auto">
          {#if fetchingLinkTree}
            <div class="flex items-center justify-center py-12">
              <LoaderCircle size={32} class="animate-spin text-amber-600" />
            </div>
          {:else}
            <div>
              <label for="edit-linktree-title-input" class="block text-sm font-semibold text-slate-500 mb-1.5">
                Judul Link Tree <span class="text-red-700 font-bold">*</span>
              </label>
              <input
                id="edit-linktree-title-input"
                type="text"
                data-testid="edit-linktree-title-input"
                bind:value={title}
                class="w-full px-4 py-3 rounded-xl border border-slate-200 bg-white text-slate-500 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 transition-all text-sm shadow-xs"
                required
              />
            </div>

            <div>
              <label for="edit-linktree-type-select" class="block text-sm font-semibold text-slate-500 mb-1.5">
                Tipe Visibilitas <span class="text-red-700 font-bold">*</span>
              </label>
              <select
                id="edit-linktree-type-select"
                data-testid="edit-linktree-type-select"
                bind:value={type}
                class="w-full px-4 py-3 rounded-xl border border-slate-200 bg-white text-slate-500 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 transition-all text-sm shadow-xs"
              >
                <option value="public">Publik (Bisa dilihat siapa saja)</option>
                <option value="unlisted">Unlisted (Hanya lewat tautan langsung)</option>
                <option value="private">Pribadi (Hanya saya)</option>
              </select>
            </div>

            <div>
              <label for="edit-linktree-description-input" class="block text-sm font-semibold text-slate-500 mb-1.5">
                Deskripsi <span class="text-red-700 font-bold">*</span>
              </label>
              <textarea
                id="edit-linktree-description-input"
                data-testid="edit-linktree-description-input"
                bind:value={description}
                rows="4"
                class="w-full px-4 py-3 rounded-xl border border-slate-200 bg-white text-slate-500 placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-600 transition-all text-sm shadow-xs resize-none"
                required
              ></textarea>
            </div>
          {/if}
        </div>

        <!-- Modal Footer -->
        <div class="flex items-center justify-end gap-3 px-6 py-4 border-t border-slate-200 bg-slate-50/80 shrink-0">
          <button
            type="button"
            data-testid="cancel-edit-modal-btn"
            on:click={handleClose}
            disabled={loading}
            class="px-5 py-2.5 text-sm font-medium text-slate-500 hover:text-slate-500 hover:bg-slate-200/60 rounded-xl transition-colors"
          >
            Batal
          </button>
          <button
            type="submit"
            data-testid="submit-edit-modal-btn"
            disabled={loading || fetchingLinkTree}
            class="inline-flex items-center gap-2 px-6 py-2.5 text-sm font-semibold text-white bg-amber-600 hover:bg-amber-700 active:bg-amber-800 rounded-xl shadow-md shadow-amber-600/25 transition-all disabled:opacity-60"
          >
            {#if loading}
              <LoaderCircle size={18} class="animate-spin" />
              <span>Menyimpan...</span>
            {:else}
              <PenLine size={18} strokeWidth={2.5} />
              <span>Perbarui Link Tree</span>
            {/if}
          </button>
        </div>
      </form>
    </div>
  </div>
{/if}
