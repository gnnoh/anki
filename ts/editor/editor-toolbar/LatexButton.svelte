<!--
Copyright: Ankitects Pty Ltd and contributors
License: GNU AGPL, version 3 or later; http://www.gnu.org/licenses/agpl.html
-->
<script lang="ts">
    import * as tr from "@tslib/ftl";
    import { getPlatformString } from "@tslib/shortcuts";
    import { wrapInternal } from "@tslib/wrap";

    import DropdownItem from "../../components/DropdownItem.svelte";
    import IconButton from "../../components/IconButton.svelte";
    import Popover from "../../components/Popover.svelte";
    import Shortcut from "../../components/Shortcut.svelte";
    import WithFloating from "../../components/WithFloating.svelte";
    import { mathjaxConfig } from "../../editable/mathjax-element";
    import { context as noteEditorContext } from "../NoteEditor.svelte";
    import type { RichTextInputAPI } from "../rich-text-input";
    import { editingInputIsRichText } from "../rich-text-input";
    import { functionIcon } from "./icons";

    const { focusedInput } = noteEditorContext.get();
    $: richTextAPI = $focusedInput as RichTextInputAPI;

    async function surround(front: string, back: string): Promise<void> {
        const element = await richTextAPI.element;
        wrapInternal(element, front, back, false);
    }

    function onMathjaxInline(): void {
        if (mathjaxConfig.enabled) {
            surround("<anki-mathjax focusonmount>", "</anki-mathjax>");
        } else {
            surround("\\(", "\\)");
        }
    }

    function onMathjaxBlock(): void {
        if (mathjaxConfig.enabled) {
            surround('<anki-mathjax block="true" focusonmount>', "</anki-matjax>");
        } else {
            surround("\\[", "\\]");
        }
    }

    function onMathjaxChemistry(): void {
        if (mathjaxConfig.enabled) {
            surround('<anki-mathjax focusonmount="0,4">\\ce{', "}</anki-mathjax>");
        } else {
            surround("\\(\\ce{", "}\\)");
        }
    }

    function onLatex(): void {
        surround("[latex]", "[/latex]");
    }

    function onLatexEquation(): void {
        surround("[$]", "[/$]");
    }

    function onLatexMathEnv(): void {
        surround("[$$]", "[/$$]");
    }

    type LatexItem = [() => void, string, string];

    const dropdownItems: LatexItem[] = [
        [onMathjaxInline, "Control+M, M", tr.editingMathjaxInline()],
        [onMathjaxBlock, "Control+M, E", tr.editingMathjaxBlock()],
        [onMathjaxChemistry, "Control+M, C", tr.editingMathjaxChemistry()],
        [onLatex, "Control+T, T", tr.editingLatex()],
        [onLatexEquation, "Control+T, E", tr.editingLatexEquation()],
        [onLatexMathEnv, "Control+T, M", tr.editingLatexMathEnv()],
    ];

    $: disabled = !$focusedInput || !editingInputIsRichText($focusedInput);

    let showFloating = false;
</script>

<WithFloating
    show={showFloating && !disabled}
    closeOnInsideClick
    inline
    on:close={() => (showFloating = false)}
>
    <IconButton
        slot="reference"
        {disabled}
        on:click={() => (showFloating = !showFloating)}
    >
        {@html functionIcon}
    </IconButton>

    <Popover slot="floating" --popover-padding-inline="0">
        {#each dropdownItems as [callback, keyCombination, label]}
            <DropdownItem on:click={() => setTimeout(callback, 100)}>
                <span>{label}</span>
                <span class="ms-auto ps-2 shortcut"
                    >{getPlatformString(keyCombination)}</span
                >
            </DropdownItem>
        {/each}
    </Popover>
</WithFloating>

{#each dropdownItems as [callback, keyCombination]}
    <Shortcut {keyCombination} on:action={callback} />
{/each}

<style lang="scss">
    .shortcut {
        font: Verdana;
    }
</style>
