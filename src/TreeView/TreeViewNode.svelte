<script context="module">
  /**
   * Computes the depth of a tree leaf node relative to <ul role="tree" />
   * @param {HTMLLIElement} node
   * @returns {number} depth
   */
  export function computeTreeLeafDepth(node) {
    let depth = 0;

    if (node == null) return depth;

    let parentNode = node.parentNode;

    while (parentNode != null && parentNode.getAttribute("role") !== "tree") {
      parentNode = parentNode.parentNode;
      if (parentNode.tagName === "LI") depth++;
    }

    return depth;
  }

  /**
   * Finds the nearest parent tree node
   * @param {HTMLElement} node
   * @returns {null | HTMLElement}
   */
  function findParentTreeNode(node) {
    if (node.classList.contains("bx--tree-parent-node")) return node;
    if (node.classList.contains("bx--tree")) return null;
    return findParentTreeNode(node.parentNode);
  }
</script>

<script>
  /**
   * @typedef {string | number} TreeNodeId
   * @slot {{ node: { id: TreeNodeId; text: string; expanded: false, leaf: boolean; disabled: boolean; selected: boolean; } }}
   */

  export let leaf = false;

  /** @type {TreeNodeId} */
  export let id = "";
  export let text = "";
  export let disabled = false;

  /**
   * Specify the icon to render
   * @type {any}
   */
  export let icon = undefined;

  import { afterUpdate, getContext } from "svelte";

  let ref = null;
  let refLabel = null;
  let prevActiveId = undefined;

  const { activeNodeId, selectedNodeIds, clickNode, selectNode, focusNode } =
    getContext("TreeView");
  const offset = () =>
    computeTreeLeafDepth(refLabel) + (leaf && icon ? 2 : 2.5);

  afterUpdate(() => {
    if (id === $activeNodeId && prevActiveId !== $activeNodeId) {
      if (!$selectedNodeIds.includes(id)) selectNode(node);
    }

    prevActiveId = $activeNodeId;
  });

  $: selected = $selectedNodeIds.includes(id);
  $: node = {
    id,
    text,
    // A node cannot be expanded.
    expanded: false,
    leaf,
    disabled,
    selected,
  };
  $: if (refLabel) {
    refLabel.style.marginLeft = `-${offset()}rem`;
    refLabel.style.paddingLeft = `${offset()}rem`;
  }
</script>

<!-- svelte-ignore a11y-no-noninteractive-element-to-interactive-role -->
<li
  bind:this={ref}
  role="treeitem"
  {id}
  tabindex={disabled ? undefined : -1}
  aria-current={id === $activeNodeId || undefined}
  aria-selected={disabled ? undefined : selected}
  aria-disabled={disabled}
  class:bx--tree-node={true}
  class:bx--tree-leaf-node={true}
  class:bx--tree-node--active={id === $activeNodeId}
  class:bx--tree-node--selected={selected}
  class:bx--tree-node--disabled={disabled}
  class:bx--tree-node--with-icon={icon}
  on:click|stopPropagation={() => {
    if (disabled) return;
    clickNode(node);
  }}
  on:keydown={(e) => {
    if (e.key === "ArrowLeft" || e.key === "ArrowRight" || e.key === "Enter") {
      e.stopPropagation();
    }

    if (e.key === "ArrowLeft") {
      const parentNode = findParentTreeNode(ref.parentNode);
      if (parentNode) parentNode.focus();
    }

    if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      if (disabled) return;
      clickNode(node);
    }
  }}
  on:focus={() => {
    focusNode(node);
  }}
>
  <div bind:this={refLabel} class:bx--tree-node__label={true}>
    <svelte:component this={icon} class="bx--tree-node__icon" />
    <slot {node}>
      {text}
    </slot>
  </div>
</li>
