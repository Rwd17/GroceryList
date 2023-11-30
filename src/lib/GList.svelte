<svelte:options immutable= {true}/>

<script>
    import {createEventDispatcher, afterUpdate} from "svelte";

    afterUpdate (() => {
    if (autoscroll) listDiv.scrollTo(0, listDivScrollHeight);
    autoscroll = false;
  });


    export let items= null;
    export let error = null;
    export let isLoading = false;
    export let disableAdding = false;
    export let disabledItems = [];


    let prevItems = items;
    let inputText = '';
    let input, listDiv, autoscroll, listDivScrollHeight;
  

    const dispatch = createEventDispatcher();

    $:{ 
        autoscroll = items && prevItems && items.length > prevItems.length;
        prevItems = items;
    }


    export function clearInput() {
		inputText = '';
	}
    export function focusInput() {
		input.focus();
	}


    function handleAddItem(){
        const isNotdeleted =dispatch('additem', {
            title: inputText
        },
        { cancelable: true }
		);
		if (isNotdeleted) {
			inputText = '';
		}
        }
    
    function handleDeleteItem(id){
        dispatch('deleteitem', {id}
            );
    }

    function handletoggleItem(id, value){
        dispatch('toggleitem', {id, value}
            );
    }


</script>


<div class="GList">
    
    {#if isLoading}
       <p class="state-text">Loading...</p>
    {:else if error}
       <p class="state-text">{error}</p>
       {:else if items}
       <div class="Item-List" bind:this={listDiv}>
         <div bind:offsetHeight={listDivScrollHeight}>
           {#if items.length === 0}
             <p class="state-text">No Items yet</p>
           {:else}
           <ul>
             {#each items as item, index (item.id)}
             {@const { id, completed, title } = item}
              <li>
                <slot {item} {index} {handletoggleItem}>
                    <div class:completed>
                  <label>
                        <input
                        disabled={disabledItems.includes(id)}
                             on:input={(event)=> {
                                  event.currentTarget.checked = completed;
                                   handletoggleItem(id, !completed)}}
                                   type="checkbox" checked={completed}/>
                                   <slot name="title">{title}</slot>
                                      </label>
                                      <button 
                                      disabled={disabledItems.includes(id)}
                                      class="Delete-item-button"
                                      aria-label="Delete item: {title}"
                                      on:click={()=>handleDeleteItem(id)}>Delete
                                      <span style:width="10px" style:display="inline-block">
                                        
                                    </span>
                                  </button>
                                </div>
                              </slot>
                            </li>
                          {/each}
                        </ul>
                      {/if}
                    </div>
                  </div>
                {/if}


<form class="add-to-list-form" on:submit|preventDefault={handleAddItem}>
    <input 
    disabled={disableAdding || !items}
    bind:this={input}
    bind:value={inputText}
    placeholder="New Todo"/>
    
    <button  class="add-item-button" type="submit"disabled={!inputText || disableAdding || !items}>Add</button>
</form>

</div>

<style>
  
  .GList {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
  }

  /* Styles for the loading/error state text */
  .state-text {
    font-size: 18px;
    color: #ff6347; /* Error text color */
    margin-top: 10px;
  }

  /* Styling for the add-to-list form */
  .add-to-list-form {
    margin-top: 20px;
  }
 
</style>
