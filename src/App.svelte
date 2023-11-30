<script>

	import GList from "./lib/GList.svelte";
    import {v4 as uuid} from "uuid";
	import { tick, onMount } from 'svelte';
	import { identity } from 'svelte/internal';
	
	
	let itemList
	let showList = true;
	let items = null;
	let error = null;
	let isLoading = false;
	let isAdding = false;
	let disabledItems = [];
	let itemCount = 0;
	let count =0;
	
	onMount(() => {
    loadItems();
  });

 		 async function loadItems() {
			 isLoading = true;
			 await fetch('https://jsonplaceholder.typicode.com/todos?_limit=10').then(async (response) => {
				if (response.ok) {
					items = await response.json();
				} else {
					error = 'An error has occurred';
				}
			});
			isLoading = false;
		}



 			 async function handleAddItem(event) {
				 event.preventDefault();
				   isAdding = true;
				     await fetch('https://jsonplaceholder.typicode.com/todos', {
						method: 'POST',
						 body: JSON.stringify({
							 title: event.detail.title,
							  completed: false
							}),
							 headers: {
								'Content-type': 'application/json; charset=UTF-8'
							}
						  }).then(async (response) => {
							if (response.ok) {
								const item = await response.json();
								items = [...items, { ...item, id: uuid() }];
								itemList.clearInput();
							} else {
								alert('An error has occurred.');
							}
						});
						isAdding = false;
						 await tick();
						 itemList.focusInput();
						}



			async function handleDeleteItem(event) {
				const id = event.detail.id;

				if (disabledItems.includes(id)) return;

				disabledItems = [...disabledItems, id];

				await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
					method: 'DELETE'
				}).then((response) => {

					if (response.ok) {
						items = items.filter((i) => i.id !== event.detail.id);
					} else {
						alert('An error has occurred.');
					 }
					 });
					  disabledItems = disabledItems.filter((itemId) => itemId !== id);
					 }



			 async function handletoggleItem(event) {

				const id = event.detail.id;
 				const value = event.detail.value;

				  if (disabledItems.includes(id)) return;

				  disabledItems = [...disabledItems, id];

				  await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
					method: 'PATCH',
					body: JSON.stringify({
						completed: value
					}),

					 headers: {
						'Content-type': 'application/json; charset=UTF-8'
					}
				 }).then(async (response) => {

					if (response.ok) {

						 const updatedItem = await response.json();
						 
						 items = items.map((item) => {
							
							if (item.id === id) {
								 return updatedItem;
								}
								return { ...item };
							 });
							} else {
								alert('Something wrong happend!');
							}
						});
						
						disabledItems = disabledItems.filter((itemId) => itemId !== id);
					}

					$: itemCount = items ? items.length : 0;

					$: countChecked =()=> {
						let count = 0;
						     if (items) {
								 items.forEach((item) => {
									      if (item.completed) {
											    count++;
											  }
											 })
											 }
											  return count;
											  }
					
					

					
</script>
<h1>Grocery List</h1>



{#if showList}
  <div class="Items">
<GList 
{items}
{error}
{isLoading}
{disabledItems}
disableAdding={isAdding} 
bind:this={itemList}
on:additem = {handleAddItem}
on:deleteitem={handleDeleteItem}
on:toggleitem={handletoggleItem}
let:item
let:handletoggleItem
let:index
>

</GList>
</div>
{/if}

<h3>Number of items: {itemCount}</h3>
<h3>number of checked {countChecked()}</h3>
<style>
 h1{
	color: rgb(44, 28, 28);
	text-align: center;
 }
 h3{
	color: rgb(44, 28, 28);
	text-align: center;
 }
</style> 