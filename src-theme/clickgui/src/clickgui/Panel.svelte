<script>
    import {sineInOut} from "svelte/easing";
    import {slide} from "svelte/transition";
    import Module from "./Module.svelte";

    export let name;
    export let modules;

    export let startTop;
    export let startLeft;

    let expanded = storage.getItem(`clickgui.panel.${name}.expanded`) === "true" || false;

    let renderedModules = expanded ? modules : [];

    let top = parseInt(storage.getItem(`clickgui.panel.${name}.top`)) || startTop;
    let left = parseInt(storage.getItem(`clickgui.panel.${name}.left`)) || startLeft;
    let moving = false;
    let prevX = 0;
    let prevY = 0;

    // Panels should not go out of bounds
    if (top < 0) {
        top = 0;
    }
    if (left < 0) {
        left = 0;
    }

	function onMouseDown() {
		moving = true;
	}
	
	function onMouseMove(e) {
		if (moving) {
			left += e.screenX - prevX;
			top += e.screenY - prevY;

            // Panels should not go out of bounds
            if (top < 0) {
                top = 0;
            }
            if (left < 0) {
                left = 0;
            }
		}

        prevX = e.screenX;
        prevY = e.screenY;
	}

	function onMouseUp() {
		moving = false;

        storage.setItem(`clickgui.panel.${name}.top`, top);
        storage.setItem(`clickgui.panel.${name}.left`, left);
	}

    function toggleExpanded(e) {
        if (expanded) {
            expanded = false;
            renderedModules = [];
        } else {
            expanded = true;
            renderedModules = modules;
        }
        storage.setItem(`clickgui.panel.${name}.expanded`, expanded);
    }

    window.addEventListener("mouseup", onMouseUp);
    window.addEventListener("mousemove", onMouseMove);

    function handleToggleModule(event) {
        const targetModule = event.getModule().getName();
        modules.find(m => m.name === targetModule).enabled = event.getNewState();
        if (expanded) {
            renderedModules = modules;
        }
    }

    function handleToggleClick(event) {
        if (event.button === 2) {
            toggleExpanded();
        }
    }

    try {
        events.on("toggleModule", handleToggleModule);
    } catch (err) {
        console.log(err);
    }
</script>

<div class="panel" style="left: {left}px; top: {top}px;">
    <div on:mousedown={handleToggleClick} on:mousedown={onMouseDown} class="title-wrapper">
        <img class="icon" src="img/{name.toLowerCase()}.svg" alt="icon" />
        <div class="title">{name}</div>
        <div on:click={toggleExpanded} class="visibility-toggle" class:expanded={expanded}></div>
    </div>
    <div class="modules">
        {#each renderedModules as m}
            <div transition:slide={{duration: 400, easing: sineInOut}}>
                <Module instance={m.instance} enabled={m.enabled} />
            </div>
        {/each}
    </div>
</div>

<style lang="scss">
    .panel {
        border-radius: 5px;
        overflow: hidden;
        width: 225px;
        position: absolute;
    }

    .title-wrapper {
        display: grid;
        grid-template-columns: max-content auto max-content;
        align-items: center;
        column-gap: 12px;
        background-color: rgba(0, 0, 0, 0.68);
        border-bottom: solid 2px #4677ff;
        padding: 10px 15px;
    }

    .title {
        font-size: 14px;
        font-weight: 600;
        color: white;
    }

    .modules {
        background-color: rgba(0, 0, 0, 0.5);
        max-height: 545px;
        overflow: auto;
    }

    .visibility-toggle {
        height: 12px;
        width: 12px;
        position: relative;

        &::before {
            content: "";
            position: absolute;
            background-color: white;
            transition: transform 0.4s ease-out;
            top: 0;
            left: 50%;
            width: 2px;
            height: 100%;
            margin-left: -1px;
        }

        &::after {
            content: "";
            position: absolute;
            background-color: white;
            transition: transform 0.4s ease-out; 
            top: 50%;
            left: 0;
            width: 100%;
            height: 2px;
            margin-top: -1px; 
        }

        &.expanded {
            &::before {
                transform: rotate(90deg);
            }

            &::after {
                transform: rotate(180deg);
            }
        }
    }

    ::-webkit-scrollbar {
        width: 0;
    }
</style>
