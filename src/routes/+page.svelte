<script>
    import { onMount, onDestroy } from 'svelte';
    import { writable } from 'svelte/store';

    const operandArr = ["÷", "x", "-", "+"];
    const dataTypes = ['header', 'number', 'comment', 'operator'];

    let calcMem = writable([]);
    let saveIco = writable('≡');
    let calcDisp = writable('');
    let calcOp = writable('');
    let delKey = writable('C');


    // handle ALL clicks here
    const handleClick = (e) => {
        const num = e.target.innerHTML;
        const numId = e.target.id;
        console.log(num, $calcDisp, $calcMem.length);

        if (!isNaN(+num) && typeof +num === 'number') {
            console.log('handle num')
            handleNumberClick(num, e);
        } else if (num === '.') {
            console.log('handle dot')
            handleDotClick();
        } else if (operandArr.includes(num) && calcDisp !== '' && numId !== 'menu-icon-place') {
            console.log('handle operand')
            handleOperandClick(num, e);
        } else if (num === '=') {
            handleEqualClick();
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

    const handleOperandClick = (num, e) => {
        if (e.target.matches('button')) {
            if ($calcDisp !== '') {
                // handle when operand hit after finished calculation
                if ($calcMem.includes('=')) {
                    calcOp.set('');
                    calcMem.update((prev) => [$calcDisp, num])
                    calcDisp.set('');
                } else {
                    // add operands to the array
                    calcOp.set(num);
                    calcMem.update((prev) => [...prev, $calcDisp, num]);
                    calcDisp.set('');
                }
            } else {
                // change the last operand in the array
                if ($calcMem.length > 0) {
                    calcOp.set(num);
                    calcMem.update((prev) => [...prev.slice(0, -1), num]);
                }
            };
        }
    };

    const parseOp = (num1, op, num2) => {
        num1 = +(num1);
        num2 = +(num2);
        if (op === '+') return num1 + num2;
        if (op === '-') return num1 - num2;
        if (op === 'x' || op === '*') return num1 * num2;
        if (op === '÷' || op === '/') return num1 / num2;
    };

    const recalc = (arr) => {
        let temp2 = [].concat(arr);
        let temp = parseOp(temp2[0], temp2[1], temp2[2]);
        console.log(temp2);
        temp2.splice(0,3)

        while(temp2.length > 0 && temp2[0] !== '=') {
            let cur = temp2.splice(0,2);
            temp = parseOp(temp, cur[0], cur[1]);
        }
        console.log(temp);
        return temp;
    };

    const handleEqualClick = () => {
        if ($calcDisp !== '' && $calcMem.length > 1) {
            let tempMem = [...$calcMem, $calcDisp];
            let tempDisp = recalc(tempMem);
            $calcMem = [...$calcMem, $calcDisp, '=', tempDisp];
            calcDisp.set(tempDisp);
            saveIco.set('<');
        }
    }

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
            <div class="calc-mem">{$calcMem.join('')}</div>
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


