<script lang="ts">
    import { onMount } from "svelte";

    import type { DirectoryNode } from "@utils/directory";


    let { tree, currentPath } = $props<{
        tree: DirectoryNode[];
        currentPath: string;
    }>();

    let expandedFolders = $state<Record<string, boolean>>({});

    function isPathActive(nodeUrl?: string): boolean {
        if (!nodeUrl || !currentPath) return false;
        const normalizedCurrent = currentPath.replace(/\/$/, "");
        const normalizedNode = nodeUrl.replace(/\/$/, "");
        return normalizedCurrent === normalizedNode;
    }

    function toggleFolder(folderPath: string, event: Event) {
        event.stopPropagation();
        expandedFolders[folderPath] = !expandedFolders[folderPath];
    }

    function autoExpand(nodes: DirectoryNode[], parentPath: string): boolean {
        let hasActiveChild = false;
        for (const node of nodes) {
            const nodePath = parentPath ? `${parentPath}/${node.name}` : node.name;
            if (node.type === "file") {
                if (isPathActive(node.url)) {
                    hasActiveChild = true;
                }
            } else if (node.type === "folder" && node.children) {
                const childActive = autoExpand(node.children, nodePath);
                if (childActive) {
                    expandedFolders[nodePath] = true;
                    hasActiveChild = true;
                }
            }
        }
        return hasActiveChild;
    }

    onMount(() => {
        autoExpand(tree, "");
    });
</script>

{#snippet renderNode(node: DirectoryNode, path: string, depth: number)}
    {@const currentPathStr = path ? `${path}/${node.name}` : node.name}
    <div class="flex flex-col">
        {#if node.type === "folder"}
            <!-- svelte-ignore a11y_click_events_have_key_events -->
            <!-- svelte-ignore a11y_no_static_element_interactions -->
            <div 
                class="flex items-center px-2 py-1.5 my-0.5 rounded-md cursor-pointer transition-colors duration-200 hover:text-(--primary) hover:bg-(--btn-plain-bg-hover) {expandedFolders[currentPathStr] ? 'font-medium text-90' : 'font-medium text-75'}" 
                onclick={(e) => toggleFolder(currentPathStr, e)}
            >
                <div class="flex items-center justify-center mr-1.5 w-4 h-4 transition-transform duration-200 {expandedFolders[currentPathStr] ? 'rotate-90' : ''}">
                    <svg width="1em" height="1em" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <polyline points="9 18 15 12 9 6"></polyline>
                    </svg>
                </div>
                <span class="overflow-hidden text-ellipsis whitespace-nowrap">{node.name}</span>
            </div>
            
            {#if expandedFolders[currentPathStr] && node.children}
                <div class="flex flex-col pl-3 ml-2 border-l border-(--line-divider)">
                    {#each node.children as child}
                        {@render renderNode(child, currentPathStr, depth + 1)}
                    {/each}
                </div>
            {/if}
        {:else}
            <a href={node.url} class="flex items-center px-2 py-1.5 my-0.5 rounded-md cursor-pointer transition-colors duration-200 hover:text-(--primary) hover:bg-(--btn-plain-bg-hover) {isPathActive(node.url) ? 'text-(--primary) bg-(--btn-plain-bg-hover) font-medium' : 'text-75'}">
                <span class="overflow-hidden text-ellipsis whitespace-nowrap">{node.name}</span>
            </a>
        {/if}
    </div>
{/snippet}

<div class="text-sm select-none pb-2">
    {#each tree as node}
        {@render renderNode(node, "", 0)}
    {/each}
</div>