<script>
    import { onMount, onDestroy } from 'svelte';
    import { writable } from 'svelte/store';

    const operandArr = ["÷", "x", "-", "+"];
    const dataTypes = ['header', 'number', 'comment', 'operator'];

    let calcMem = writable([]);
    let menuIcon = writable('≡');
    let saveIco = writable('');
    let calcDisp = writable('');
    let calcOp = writable('');
    let delKey = writable('C');

    let calcStorage = writable(() => {
        const item = localStorage.getItem('Calc_save');
        return JSON.parse(item) || {};
    })

    let countInit = () => {
        let temp = Object.keys($calcStorage);
        return (temp.length > 0) ? +(temp[temp.length - 1].match(/\d+/)[0]) + 1 : 0;
    };

    let calcMemCount = writable(countInit());

    // handle ALL clicks here
    const handleClick = (e) => {
        const num = e.target.innerHTML;
        const numId = e.target.id;
        let dataT = e.target.dataset;
        let type = e.target.dataset.type;
        console.log(num, $calcDisp, $calcMem.length, type);

        if (!isNaN(+num) && typeof +num === 'number') {
            console.log('handle num')
            handleNumberClick(num, e);
        } else if (num === '.') {
            console.log('handle dot')
            handleDotClick();
        } else if (operandArr.includes(num) && calcDisp !== '' && numId !== 'menu-icon-place' && type !== 'deleteButton') {
            console.log('handle operand')
            handleOperandClick(num, e);
        } else if (num === '=') {
            handleEqualClick();
        } else if (num === 'C' || num === 'CE') {
            handleDel(num);
        } else if (numId === 'save-icon-place') {
            saveCalc();
        } else if (numId === 'menu-icon-place') {
            handleMenu(e);
        } else if (type === 'deleteButton') {
            handleDelete(e);
        } else if (type === 'addNum') {
            addNum(e);
        }
    };

    // All functions from handle All clicks
    const handleNumberClick = (num, e) => {
        if (e.target.matches('button')) {
            // console.log('it is a num button')
            if (!$calcMem.includes('=')) {
                // concat number to the existing text variable
                calcDisp.update((prevDisp) => prevDisp + num);
                if ($calcOp !== '') {
                    calcOp.set('');
                }
                if ($delKey === 'CE') {
                    delKey.set('C');
                }
            } else {
                if ($delKey === 'CE') {
                    delKey.set('C');
                }
                calcDisp.set('');
                $calcMem = [];
            };
        }
    };

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
    };

    const handleDel = (operand) => {
        if (operand === 'C' && $calcDisp !== '') {
            calcDisp.set('');
            delKey.set($calcMem.length > 0 ? 'CE' : 'C');
        } else if (operand === 'CE') {
            if ($calcDisp !== '') {
                calcDisp.set('');
                delKey.set('CE');
            } else {
                $calcMem = [];
                delKey.set('C');
            };
        }
    };

    const saveCalc = () => {
        let name = 'calc_' + $calcMemCount;
        let newCalc = $calcMem.map(a => (typeof +a === 'number' && !isNaN(a)) ? +a : a);

        calcStorage.update((prev) => ({
            ...prev,
            [name]: {
                'calculation': newCalc,
                'comments': newCalc.slice(0, -2).map(a => (typeof a == 'number' && a !== '=') ? '...' : null).concat(null, null),
                'name': name
            }
        }));

        calcMemCount.set($calcMemCount + 1);
        saveIco.set('');
        $calcMem = [];
        calcDisp.set('');
        console.log($calcStorage);
    };

    const handleMenu = (e) => {
        let sel = e.target.innerHTML;
        let sel2 = document.querySelector('.calc-mem-storage');
        if (sel === '≡') {
            menuIcon.set('x');
        }

        if (sel === 'x') {
            menuIcon.set('≡');
        }

        let condi = sel2 !== null;
        if (condi) return sel2.classList.toggle('visible');
    };

    const reindexKeys = (obj) => {
      const currKeys = Object.keys(obj);
      const mapping = currKeys.reduce((acc, key, index) => {
        const newKey = `calc_${index}`;
        acc[key] = newKey;
        return acc;
      }, {});
      const updatedObj = Object.fromEntries(
        Object.entries(obj).map(([oldKey, value]) => {
            const newKey = mapping[oldKey];
            const currentName = value.name;

            const updatedName = currentName.includes('calc_') ? newKey : currentName;

            const updatedValue = {
            ...value,
            name: updatedName,
            };
            return [newKey, updatedValue];
        })
      );
      return updatedObj;
    };

    const handleDelete = (e) => {
        console.log('deletre')
        let parent = e.target.dataset.idparent;

        calcStorage.update((prev) => {
        const entries = Object.entries(prev);
        const filtered = entries.filter(([key]) => key !== parent);
        const newObj = Object.fromEntries(filtered);
        return reindexKeys(newObj);
        });

        $calcMemCount = +Object.keys($calcStorage).pop().replace(/\D+/, '') + 1;
    };

    const addNum = (e) => {
      const key = e.target.dataset.idparent;

      const updatedCalc = [...$calcStorage[key]['calculation'].slice(0, -2), '+', 0, ...$calcStorage[key]['calculation'].slice(-2)];
      const updatedComm = [...$calcStorage[key]['comments'].slice(0, -2), null, '...', null, null];

      $calcStorage[key] = {
        ...$calcStorage[key],
        calculation: updatedCalc,
        comments: updatedComm
      };
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

<div class='container'>
    <div id='calc-main' class='calc-grid'>
        <div class="calc-display">
            <div id="menu-icon-place" class="storage-window">{$menuIcon}</div>
            <div class="calc-current">{$calcDisp}</div>
            <div class="calc-mem">{$calcMem.join('')}</div>
            <div><span class="save-button saveIcon" id="save-icon-place">{$saveIco}</span></div>
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
    <div id="calc-mem-storage" class="calc-mem-storage">
        {#each Object.entries($calcStorage) as [key, { calculation, comments, name }]}
          <div class='storage-item' key={key} id={key}>
            <div>
              <h3>
                <span class="editable" data-type="header" data-idparent={key}>{name}</span>
                <button class="delete-mem" data-type="deleteButton" data-idparent={key}>x</button>
              </h3>
            </div>
            {#each calculation as a, b}
              {#if typeof a === 'number'}
                {#if b === calculation.length - 1}
                  <div key={b}>
                    <strong>
                      <span data-type="number" data-idparent={key} data-index={b}>{a}</span>
                    </strong>
                    <button data-type="addNum" data-index={b} data-idparent={key}>Add</button>
                  </div>
                {:else}
                  <div key={b}>
                    <strong>
                      <span class="editable" data-type="number" data-idparent={key} data-index={b}>{a}</span>
                    </strong>
                    <span class="editable" data-type="comment" data-idparent={key} data-index={b}>{comments[b]}</span>
                  </div>
                {/if}
              {:else}
                <span key={b} class="editable" data-type="operator" data-idparent={key} data-index={b}>{a}</span>
              {/if}
            {/each}
          </div>
        {/each}
      </div>
</div>


