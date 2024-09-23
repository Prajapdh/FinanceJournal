<script>
    import { Button, buttonVariants } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Tabs, TabsContent, TabsList, TabsTrigger } from "$lib/components/ui/tabs";
    import * as Card from "$lib/components/ui/card";
    import { Textarea } from "$lib/components/ui/textarea";
    import { Alert, AlertDescription, AlertTitle } from "$lib/components/ui/alert";
    import { Label } from "$lib/components/ui/label";
    import { writable } from 'svelte/store';
    import * as Dialog from "$lib/components/ui/dialog";
    import { ChevronRight } from 'lucide-svelte';
    import { Progress } from "$lib/components/ui/progress";
    
    // Assuming you have a way to control the active tab
    let activeTab = 'dashboard';
    function setActiveTab(tab) {
        activeTab = tab;
    }
  
    // Type definitions (for clarity, but not enforced in JS)
    // Transaction type
    // type Transaction = { id: number, description: string, amount: number, category: string, notes: string, date: string };
    
    // Reminder type
    // type Reminder = { id: number, text: string, date: string };
    
    // Budget type
    // type Budget = { id: number, category: string, amount: number };
    
    // Goal type
    // type Goal = { id: number, description: string, targetAmount: number, currentAmount: number };

    // Stores
    let transactions = writable([]);
    let reminders = writable([]);
    let budgets = writable([]);
    let goals = writable([]);
    let selectedGoal = null; // Goal or null
    let selectedBudget = null; // Budget or null

    // Form variables
    let description = '';
    let amount = 0;
    let transactionCategory = '';
    let notes = '';
    let reminderText = '';
    let reminderDate = '';
    let budgetCategory = '';
    let budgetAmount = 0;
    let goalDescription = '';
    let goalTargetAmount = 0;
    let goalCurrentAmount = 0;

    // Editable fields for budget form
    let editBudgetCategory = '';
    let editBudgetAmount = 0;

    // When selectedBudget is set, update the local variables
    $: if (selectedBudget) {
        editBudgetCategory = selectedBudget.category;
        editBudgetAmount = selectedBudget.amount;
    }
    
    // Editable fields for goal form
    let editGoalDescription = '';
    let editGoalTargetAmount = 0;
    let editGoalCurrentAmount = 0;

    function addTransaction() {
        console.log("In addTransaction, transactionCategory: ", transactionCategory);
        transactions.update(txs => [
            { id: Date.now(), description, amount, category: transactionCategory, notes, date: new Date().toISOString() },
            ...txs
        ]);
        // Reset form fields
        description = '';
        amount = 0;
        transactionCategory = '';
        notes = '';
    }

    function deleteTransaction(id) {
        transactions.update(txs => txs.filter(t => t.id !== id));
    }

    function addReminder() {
        reminders.update(rs => [
            { id: Date.now(), text: reminderText, date: reminderDate },
            ...rs
        ]);
        reminderText = '';
        reminderDate = '';
    }

    function deleteReminder(id) {
        reminders.update(rs => rs.filter(r => r.id !== id));
    }

    function addBudget() {
        budgets.update(bs => [
            { id: Date.now(), category: budgetCategory, amount: budgetAmount },
            ...bs
        ]);
        budgetCategory = '';
        budgetAmount = 0;
    }

    function editBudget(budget) {
        selectedBudget = { ...budget };
    }

    function saveBudget(editBudgetCategory, editBudgetAmount) {
        if (selectedBudget) {
            budgets.update(bs => bs.map(b => 
                b.id === selectedBudget.id ? { ...selectedBudget, category: editBudgetCategory, amount: editBudgetAmount } : b
            ));
        }
        selectedBudget = null; // Reset selected budget after saving
    }

    function deleteBudget(id) {
        budgets.update(bs => bs.filter(b => b.id !== id));
    }

    function addGoal() {
        goals.update(gs => [
            { id: Date.now(), description: goalDescription, targetAmount: goalTargetAmount, currentAmount: goalCurrentAmount },
            ...gs
        ]);
        goalDescription = '';
        goalTargetAmount = 0;
        goalCurrentAmount = 0;
    }

    function editGoal(goal) {
        selectedGoal = { ...goal };
    }

    function saveGoal(goalDescription, goalTargetAmount, goalCurrentAmount) {
        if (selectedGoal) {
            goals.update(gs => gs.map(g => g.id === selectedGoal.id ? { ...selectedGoal, description: goalDescription, targetAmount: goalTargetAmount, currentAmount: goalCurrentAmount } : g));
        }
        selectedGoal = null; // Reset selected goal after saving
    }

    function deleteGoal(id) {
        goals.update(gs => gs.filter(g => g.id !== id));
    }

    // Function to format currency
    function formatCurrency(value) {
        return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value);
    }

    // Computed value for total spent
    $: totalSpent = $transactions.reduce((sum, t) => Number(sum) + Number(t.amount), 0.0);
</script>



<main class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-6">Finance Journal</h1>

    <Tabs class="w-full" bind:value={activeTab}>
        <TabsList>
            <TabsTrigger value="dashboard">Dashboard</TabsTrigger>
            <TabsTrigger value="transactions">Transactions</TabsTrigger>
            <TabsTrigger value="budget">Budget</TabsTrigger>
            <TabsTrigger value="goals">Goals</TabsTrigger>
            <TabsTrigger value="reminders">Reminders</TabsTrigger>
        </TabsList>

        <TabsContent value="dashboard">
			<div class="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
				<Card.Root>
					<Card.Header>
						<Card.Title>Financial Overview</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="text-2xl font-bold">{formatCurrency(totalSpent)}</div>
						<p class="text-xs text-muted-foreground">Total Spent</p>
						<div class="mt-4">
							<p class="text-sm font-medium">Transactions: {$transactions.length}</p>
						</div>
					</Card.Content>
					<Card.Footer>
						<Button variant="outline" class="w-full" on:click={() => setActiveTab('transactions')}>
							Add Transaction
							<ChevronRight class="ml-2 h-4 w-4" />
						</Button>
					</Card.Footer>
				</Card.Root>
		
				<Card.Root>
					<Card.Header>
						<Card.Title>Budgets</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="space-y-4">
							{#each $budgets.slice(0, 3) as budget}
								<div class="flex items-center">
									<div class="text-sm font-medium">
										<div>{budget.category}</div>
									</div>
										<Progress value={
											budget.amount > 0 
											? ($transactions.filter(t => t.category === budget.category)
												.reduce((sum, t) => sum + Number(t.amount), 0) / budget.amount) * 100 
											: 0} />
									<div class="ml-4 text-right">
										<p class="text-sm font-medium">{formatCurrency(budget.amount)}</p>
									</div>
								</div>
							{/each}
						</div>
					</Card.Content>
					<Card.Footer>
						<Button variant="outline" class="w-full" on:click={() => setActiveTab('budget')}>
							Manage Budgets
							<ChevronRight class="ml-2 h-4 w-4" />
						</Button>
					</Card.Footer>
				</Card.Root>
		
				<Card.Root>
					<Card.Header>
						<Card.Title>Goals</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="space-y-4">
							{#each $goals.slice(0, 3) as goal}
								<div>
									<div class="flex justify-between text-sm font-medium">
										<div>{goal.description}</div>
										<div>{formatCurrency(goal.currentAmount)} / {formatCurrency(goal.targetAmount)}</div>
									</div>
									<Progress value={(goal.currentAmount / goal.targetAmount) * 100} />
								</div>
							{/each}
						</div>
					</Card.Content>
					<Card.Footer>
						<Button variant="outline" class="w-full" on:click={() => setActiveTab('goals')}>
							View All Goals
							<ChevronRight class="ml-2 h-4 w-4" />
						</Button>
					</Card.Footer>
				</Card.Root>
		
				<Card.Root>
					<Card.Header>
						<Card.Title>Recent Transactions</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="space-y-4">
							{#each $transactions.slice(0, 5) as transaction}
								<div class="flex justify-between items-center">
									<div>
										<p class="text-sm font-medium">{transaction.description}</p>
										<p class="text-xs text-muted-foreground">{transaction.category}</p>
									</div>
									<div class="text-sm font-medium">
										{formatCurrency(transaction.amount)}
									</div>
								</div>
							{/each}
						</div>
					</Card.Content>
					<Card.Footer>
						<Button variant="outline" class="w-full" on:click={() => setActiveTab('transactions')}>
							View All Transactions
							<ChevronRight class="ml-2 h-4 w-4" />
						</Button>
					</Card.Footer>
				</Card.Root>
		
				<Card.Root>
					<Card.Header>
						<Card.Title>Upcoming Reminders</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="space-y-4">
							{#each $reminders.slice(0, 3) as reminder}
								<div class="flex justify-between items-center">
									<div>
										<p class="text-sm font-medium">{reminder.text}</p>
										<p class="text-xs text-muted-foreground">{new Date(reminder.date).toLocaleDateString()}</p>
									</div>
								</div>
							{/each}
						</div>
					</Card.Content>
					<Card.Footer>
						<Button variant="outline" class="w-full" on:click={() => setActiveTab('reminders')}>
							Manage Reminders
							<ChevronRight class="ml-2 h-4 w-4" />
						</Button>
					</Card.Footer>
				</Card.Root>
			</div>
		</TabsContent>

        <TabsContent value="transactions">
            <Card.Root>
                <Card.Header>
                    <Card.Title>Transactions</Card.Title>
                    <Card.Description>Log and view your financial activities</Card.Description>
                </Card.Header>
                <Card.Content>
                    <form class="space-y-4" on:submit|preventDefault={addTransaction}>
                        <div>
                            <Label for="description">Description</Label>
                            <Input id="description" type="text" bind:value={description} required />
                        </div>
                        <div>
                            <Label for="amount">Amount</Label>
                            <Input id="amount" type="number" bind:value={amount} required />
                        </div>
                        <div>
                            <Label for="category">Category</Label>
                            <Input id="category" type="text" bind:value={transactionCategory} required />
                        </div>
                        <div>
                            <Label for="notes">Emotional Spending Notes</Label>
                            <Textarea id="notes" bind:value={notes} />
                        </div>
                        <Button type="submit">Add Transaction</Button>
                    </form>
                    <div class="mt-8">
                        <h3 class="text-xl font-semibold mb-4">Recent Transactions</h3>
                        {#if $transactions.length === 0}
                            <p>No transactions recorded yet.</p>
                        {:else}
                            <ul class="space-y-4">
                                {#each $transactions as transaction (transaction.id)}
                                    <li key={transaction.id} class="border p-4 rounded-md flex justify-between items-start">
                                        <div>
                                            <p class="font-semibold">{transaction.description}</p>
                                            <p>{formatCurrency(transaction.amount)} - {transaction.category}</p>
                                            <p class="text-sm text-gray-500">{new Date(transaction.date).toLocaleString()}</p>
                                            {#if transaction.notes}
                                                <p class="mt-2 text-sm italic">"{transaction.notes}"</p>
                                            {/if}
                                        </div>
                                        <Button variant="destructive" on:click={() => deleteTransaction(transaction.id)}>Delete</Button>
                                    </li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                </Card.Content>
            </Card.Root>
        </TabsContent>

        <TabsContent value="budget">
            <Card.Root>
                <Card.Header>
                    <Card.Title>Budget Tracker</Card.Title>
                    <Card.Description>Manage and monitor your budget</Card.Description>
                </Card.Header>
                <Card.Content>
                    <form class="space-y-4" on:submit|preventDefault={addBudget}>
                        <div>
                            <Label for="budgetCategory">Category</Label>
							<Input id="budgetCategory" type="text" bind:value={budgetCategory} placeholder="Enter category..." required />
                        </div>
                        <div>
                            <Label for="budgetAmount">Amount</Label>
                            <Input id="budgetAmount" type="number" bind:value={budgetAmount} placeholder="Enter amount..." required />
                        </div>
                        <Button type="submit">Add Budget</Button>
                    </form>
					

                    <div class="mt-8">
                        <h3 class="text-xl font-semibold mb-4">Current Budgets</h3>
                        {#if $budgets.length === 0}
                            <p>No budgets set yet.</p>
                        {:else}
                            <ul class="space-y-4">
                                {#each $budgets as budget (budget.id)}
                                    <li key={budget.id} class="border p-4 rounded-md flex justify-between items-center">
                                        <div>
                                            <p class="font-semibold">{budget.category}</p>
                                            <p>{formatCurrency(budget.amount)}</p>
                                        </div>
                                        <div>
                                            <Dialog.Root>
                                                <Dialog.Trigger class={buttonVariants({ variant: "secondary" })} on:click={()=>{editBudget(budget); console.log(selectedBudget.category)}}>Edit Budget</Dialog.Trigger>
                                                <Dialog.Content>
													<Dialog.Header>
														<Dialog.Title>Edit Budget</Dialog.Title>
														<Dialog.Description>Update the details of your budget</Dialog.Description>
													</Dialog.Header>
												
													<div class="space-y-4 mt-4">
														<Label for="editBudgetCategory">Category</Label>
														<Input id="editBudgetCategory" type="text" bind:value={editBudgetCategory} placeholder="Enter category..." required/>
												
														<Label for="editBudgetAmount">Amount</Label>
														<Input id="editBudgetAmount" type="number" bind:value={editBudgetAmount} placeholder="Enter amount..." required/>
													</div>
												
													<Dialog.Footer>
														<Dialog.Close class={buttonVariants({ variant: "secondary" })} on:click={saveBudget(editBudgetCategory, editBudgetAmount)}>Save Changes</Dialog.Close>
														<Dialog.Close class={buttonVariants({ variant: "outline" })} on:click={() => { selectedBudget = null; }}>Cancel</Dialog.Close>
													</Dialog.Footer>
												</Dialog.Content>												
                                            </Dialog.Root>
                                            <Button variant="destructive" on:click={() => deleteBudget(budget.id)}>Delete</Button>
                                        </div>
                                    </li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                </Card.Content>
            </Card.Root>
        </TabsContent>

        <TabsContent value="goals">
            <Card.Root>
                <Card.Header>
                    <Card.Title>Savings Goals</Card.Title>
                    <Card.Description>Track your progress towards financial goals</Card.Description>
                </Card.Header>
                <Card.Content>
                    <form class="space-y-4" on:submit|preventDefault={addGoal}>
                        <div>
                            <Label for="goalDescription">Goal Description</Label>
                            <Input id="goalDescription" type="text" bind:value={goalDescription} required />
                        </div>
                        <div>
                            <Label for="goalTargetAmount">Target Amount</Label>
                            <Input id="goalTargetAmount" type="number" bind:value={goalTargetAmount} required />
                        </div>
                        <div>
                            <Label for="goalCurrentAmount">Current Amount</Label>
                            <Input id="goalCurrentAmount" type="number" bind:value={goalCurrentAmount} required />
                        </div>
                        <Button type="submit">Add Goal</Button>
                    </form>

                    <div class="mt-8">
                        <h3 class="text-xl font-semibold mb-4">Current Goals</h3>
                        {#if $goals.length === 0}
                            <p>No goals set yet.</p>
                        {:else}
                            <ul class="space-y-4">
                                {#each $goals as goal (goal.id)}
                                    <li key={goal.id} class="border p-4 rounded-md flex justify-between items-center">
                                        <div>
                                            <p class="font-semibold">{goal.description}</p>
                                            <p>Target: {formatCurrency(goal.targetAmount)}</p>
                                            <p>Current: {formatCurrency(goal.currentAmount)}</p>
                                            <p>Progress: {((goal.currentAmount / goal.targetAmount) * 100).toFixed(2)}%</p>
                                        </div>
                                        <div>
											<Dialog.Root>
												<Dialog.Trigger class={buttonVariants({ variant: "secondary" })} on:click={editGoal(goal)}>Edit Goal</Dialog.Trigger>
												<Dialog.Content>
													<Dialog.Header>
														<Dialog.Title>Edit Goal</Dialog.Title>
														<Dialog.Description>Update your goal details</Dialog.Description>
													</Dialog.Header>
												
													<div class="space-y-4 mt-4">
														<Label for="editGoalDescription">Description</Label>
														<Input id="editGoalDescription" type="text" bind:value={editGoalDescription} placeholder="Enter description..." required/>
												
														<Label for="editGoalTargetAmount">Target Amount</Label>
														<Input id="editGoalTargetAmount" type="number" bind:value={editGoalTargetAmount} placeholder="Enter target amount..." required/>
												
														<Label for="editGoalCurrentAmount">Current Amount</Label>
														<Input id="editGoalCurrentAmount" type="number" bind:value={editGoalCurrentAmount} placeholder="Enter current amount..."/>
													</div>
												
													<Dialog.Footer>
														<Dialog.Close class={buttonVariants({ variant: "secondary" })}
																	on:click={saveGoal(editGoalDescription, editGoalTargetAmount, editGoalCurrentAmount)}>
																Save Changes</Dialog.Close>
														<Dialog.Close class={buttonVariants({ variant: "outline" })} 
																	on:click={() => { selectedBudget = null; }}>
																Cancel</Dialog.Close>
													</Dialog.Footer>
												</Dialog.Content>												
											</Dialog.Root>
                                            <Button variant="destructive" on:click={() => deleteGoal(goal.id)}>Delete</Button>
                                        </div>
                                    </li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                </Card.Content>
            </Card.Root>
        </TabsContent>

        <TabsContent value="reminders">
            <Card.Root>
                <Card.Header>
                    <Card.Title>Reminders</Card.Title>
                    <Card.Description>Set reminders for upcoming payments or financial tasks</Card.Description>
                </Card.Header>
                <Card.Content>
                    <form class="space-y-4" on:submit|preventDefault={addReminder}>
                        <div>
                            <Label for="reminderText">Reminder Text</Label>
                            <Input id="reminderText" type="text" bind:value={reminderText} required />
                        </div>
                        <div>
                            <Label for="reminderDate">Date</Label>
                            <Input id="reminderDate" type="date" bind:value={reminderDate} required />
                        </div>
                        <Button type="submit">Add Reminder</Button>
                    </form>

                    <div class="mt-8">
                        <h3 class="text-xl font-semibold mb-4">Upcoming Reminders</h3>
                        {#if $reminders.length === 0}
                            <p>No reminders set yet.</p>
                        {:else}
                            <ul class="space-y-4">
                                {#each $reminders as reminder (reminder.id)}
                                    <li key={reminder.id} class="border p-4 rounded-md flex justify-between items-center">
                                        <div>
                                            <p class="font-semibold">{reminder.text}</p>
                                            <p>{new Date(reminder.date).toLocaleDateString()}</p>
                                        </div>
                                        <Button variant="destructive" on:click={() => deleteReminder(reminder.id)}>Delete</Button>
                                    </li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                </Card.Content>
            </Card.Root>
        </TabsContent>
    </Tabs>
</main>
