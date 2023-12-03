<script>
    import { onMount, onDestroy } from 'svelte';
    import { writable } from 'svelte/store';

    const operandArr = ["÷", "x", "-", "+"];
    const dataTypes = ['header', 'number', 'comment', 'operator'];

    let calcMem = writable([]);
    let calcDisp = writable('');
    let delKey = writable('C');


    // handle ALL clicks here
    const handleClick = (e) => {
        const num = e.target.innerHTML;
        console.log(num);

        if (!isNaN(+num) && typeof +num === 'number') {
            handleNumberClick(num, e);
        } else if (num === '.') {
            handleDotClick();
        }
    }

    // All functions from handle All clicks
    const handleNumberClick = (num, e) => {
        if (e.target.matches('button')) {
            // console.log('it is a num button')
            if (!$calcMem.includes('=')) {
                // concat number to the existing text variable
                calcDisp.update((prevDisp) => prevDisp + num);

            }
        }
    }

    const handleDotClick = () => {
        if (!$calcDisp.includes('.')) {
            calcDisp.update((prevDisp) => prevDisp + '.');
        }
    };

    // lifecycle function for when the component is firstly mounted
    onMount(() => {
        //event listeners
        const sel = document.querySelector('.container');
        sel.addEventListener('click', handleClick);

        // cleanup
        return () => {
            sel.removeEventListener('click', handleClick);
        };
    });
    
</script>
<slot />

<div class='container'>
    <div id='calc-main' class='calc-grid'>
        <div class="calc-display">
            <div id="menu-icon-place" class="storage-window"></div>
            <div class="calc-current">{$calcDisp}</div>
            <div class="calc-mem">{$calcMem}</div>
            <div><span class="save-button saveIcon" id="save-icon-place"></span></div>
        </div>
        <div class='calc-keys'>
            <button class="op1">+</button>
            <button class="op2">-</button>
            <button class="op3">x</button>
            <button class="op4">÷</button>
            <button>7</button>
            <button>8</button>
            <button>9</button>
            <button>4</button>
            <button>5</button>
            <button>6</button>
            <button>1</button>
            <button>2</button>
            <button>3</button>
            <button>0</button>
            <button class="op5">.</button>
            <button id="AC-key" class="op6">{$delKey}</button>
            <button class="op7">=</button>
        </div>

    </div>

    <!-- here comes the storage -->
</div>


