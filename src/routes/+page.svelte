<script>
    import { onMount, onDestroy } from 'svelte';
    import { writable } from 'svelte/store';

    // handle clicks here
    // lifecycle function for when the component is firstly mounted
    onMount(() => {
        const handleClick = (e) => {
            const num = e.target.innerHTML;
            console.log(num);

            if (!isNaN(+num) && typeof +num === 'number') {
                handleNumberClick(num, e);
            }
        }

        const handleNumberClick = (num, e) => {
            if (e.target.matches('button')) {
                console.log('it is a num button')
            }
        }

        //event listeners
        const sel = document.querySelector('.container');
        sel.addEventListener('click', handleClick);

        // lifecycle function to unmount the component
        onDestroy(() => {
            sel.removeEventListener('click', handleClick);
        });

        // cleanup
        return () => {};
    });
</script>

<div class='container'>
    <div id='calc-main' class='calc-grid'>
        <div class="calc-display">
            <div id="menu-icon-place" class="storage-window"></div>
            <div class="calc-current"></div>
            <div class="calc-mem"></div>
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
            <button id="AC-key" class="op6">C</button>
            <button class="op7">=</button>
        </div>
    </div>

    <!-- here comes the storage -->
</div>


