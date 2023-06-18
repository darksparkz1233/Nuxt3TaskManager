<script setup lang="ts">
import { Column } from '~/types';
import { Task } from '~/types';
import { nanoid } from 'nanoid'
import TrelloBoardTask from './trelloBoardTask.vue';

import draggable from 'vuedraggable'

const columns = useLocalStorage<Column[]>("trelloBoard", [
    {
        id: nanoid(),
        title: "Backlog",
        tasks: [
            {
                id: nanoid(),
                title: 'Create marketing landing page',
                createdAt: new Date,
            },
            {
                id: nanoid(),
                title: 'Develop cool new feature',
                createdAt: new Date,
            },
            {
                id: nanoid(),
                title: 'Fix optimization bugs',
                createdAt: new Date,
            },
        ]
    },
    { title: "Selected for Dev", id: nanoid(), tasks: [{ id: nanoid(), title: 'Develop a feature', createdAt: new Date }] },
    { title: "In Progress", id: nanoid(), tasks: [] },
    { title: "QA", id: nanoid(), tasks: [] },
    { title: "Completed", id: nanoid(), tasks: [] },
])

const alt = useKeyModifier("Alt")

function createColumn() {
    const column: Column = {
        id: nanoid(),
        title: "",
        tasks: []
    }

    columns.value.push(column)

    nextTick(() => {
        (
            document.querySelector(
                ".column:last-of-type .title-input"
            ) as HTMLInputElement
        ).focus();
    })
}
</script>

<template>
    <div class="flex h-96 items-start overflow-x-auto gap-4">
        <!-- Create draggable columns -->
        <draggable 
            v-model="columns" 
            group="columns" 
            :animation="200" 
            handle=".drag-handle" 
            item-key="id"
            class="flex gap-4 items-start"
        >
            <template #item="{ element: column }: { element: Column }">
                <div class="column bg-gray-200 shadow-2xl p-5 rounded-xl min-w-[300]">
                    <header class="font-bold mb-4">
                        <DragHandle />
                        <input
                            class="title-input bg-transparent focus:bg-white rounded px-1 w-4/5"
                            @keyup.enter="($event.target as HTMLInputElement).blur()" 
                            @keydown.delete="
                                column.title === '' 
                                    ? columns = columns.filter(c => c.id !== column.id) 
                                    : null
                            "
                            type="text"
                            v-model="column.title"
                        />
                    </header>
                    <!-- Create draggable tasks -->
                    <draggable 
                        v-model="column.tasks" 
                        :group="{ name: 'tasks', pull: alt ? 'clone' : true }"
                        :animation="200" 
                        handle=".drag-handle" 
                        item-key="id"
                    >
                        <template #item="{ element: task } : { element: Task }">
                            <div>
                                <TrelloBoardTask 
                                    :task="task" 
                                    @delete="column.tasks = column.tasks.filter(t => t.id !== $event)" 
                                />
                            </div>
                        </template>
                    </draggable>

                    <footer>
                        <NewTask @add="column.tasks.push($event)" />
                    </footer>
                </div>
            </template>
        </draggable>
        <button 
            @click="createColumn" 
            class="bg-gray-200 whitespace-nowrap p-2 rounded opacity-50"
        >
        + Add Another Column
        </button>
    </div>
</template>