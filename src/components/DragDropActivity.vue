<template>
	<div class="app" ref="dnd">
		<div aria-live="polite" class="sr-only" ref="liveRegion"></div>
		<div
			class="uncategorized"
			ref="uncategorized"
			@drop="drop($event, $refs.uncategorized)"
		>
			<h2>Items to Categorize</h2>
			<ul ref="uncategorizedRef">
				<li
					v-for="note in uncategorizedNotes"
					:key="note.id"
					tabindex="0"
					:data-note-id="note.id"
					data-category-name="uncategorized"
					class="note"
					:class="{ selected: note.selected }"
					draggable="true"
					@dragstart="dragStart($event, note)"
					@dragend="dragEnd($event)"
					@keydown="handleKeydown"
					@click="selectNote(note)"
					@touchstart="touchStart($event, note)"
					@touchmove="touchMove($event)"
					@touchend="touchEnd($event)"
				>
					<span>{{ note.text }}</span>
				</li>
			</ul>
		</div>
		<div class="categories">
			<div
				v-for="(category, index) in categories"
				@drop="drop($event, category)"
				:key="index"
				class="category"
				:class="category.name.toLowerCase()"
				ref="category"
				@dragover.prevent
				@click="clickCategory($event)"
				@mouseover="hoverCategory($event, true)"
				@mouseout="hoverCategory($event, false)"
				:data-category-name="category.name.toLowerCase()"
				@touchmove="touchMove($event)"
			>
				<h2>{{ category.name }}</h2>
				<ul>
					<li
						v-for="note in category.notes"
						:key="note.id"
						tabindex="0"
						:data-note-id="note.id"
						class="note"
						:class="{ selected: note.selected }"
						draggable="true"
						@dragstart="dragStart($event, note)"
						@dragend="dragEnd($event)"
						@keydown="handleKeydown"
						@click="selectNote(note)"
						@touchstart="touchStart($event, note)"
						@touchmove="touchMove($event)"
						@touchend="touchEnd($event)"
					>
						<span>{{ note.text }}</span>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>
import { nextTick } from "vue";
export default {
	props: ["data_uncategorizedNotes", "data_categories"],
	data() {
		return {
			focusedNoteId: null,
			selectedNoteId: null,
			draggedNoteId: null,
			uncategorizedNotes:this.data_uncategorizedNotes,
			categories: this.data_categories

		};
	},
	methods: {
		announceCategoryChange(noteId, categoryName) {
			const note = this.findNoteById(noteId);
			const message = `Note "${note.text}" moved to ${categoryName}.`;
			this.$refs.liveRegion.textContent = message;
		},

		async handleKeydown(event) {
			const noteId = parseInt(event.target.getAttribute("data-note-id"), 10);
			const focusedNote =
				this.uncategorizedNotes.find((note) => note.id === noteId) ||
				this.categories
					.flatMap((category) => category.notes)
					.find((note) => note.id === noteId);

			// if (!focusedNote) return;

			if (event.key === "Enter" || event.key === " ") {
				event.preventDefault();

				// If no note is being dragged, start dragging the focused note
				if (this.draggedNoteId === null) {
					this.deselectAllNotes();
					this.draggedNoteId = noteId;
					focusedNote.selected = true;
				} else {
					// If a note is being dragged, drop it
					const categoryName = event.target
						.closest(".category")
						?.getAttribute("class")
						?.split(" ")[1];
					if (categoryName) {
						this.dropNote(categoryName);
						let showName = event.target.closest(".category");
						this.announceCategoryChange(
							noteId,
							showName.getAttribute("data-category-name")
						);
					} else {
						// If dropped in the uncategorized area, move the note back to the uncategorized notes
						const noteToMove = this.categories
							.flatMap((category) => category.notes)
							.find((note) => note.id === this.draggedNoteId);

						if (noteToMove) {
							this.uncategorizedNotes.push(noteToMove);

							this.categories.forEach((category) => {
								category.notes = category.notes.filter(
									(note) => note.id !== noteToMove.id
								);
							});
						}
					}

					this.draggedNoteId = null;
					focusedNote.selected = false;

					// Wait for the next DOM update cycle before updating the focus
					await nextTick();

					// Refocus the first uncategorized note
					const firstUncategorizedNoteElement =
						this.$refs.uncategorizedRef.querySelector("li");
					if (firstUncategorizedNoteElement) {
						firstUncategorizedNoteElement.focus();
					}
				}
			} else if (event.key === "ArrowRight") {
				this.moveNote("right", noteId);
			} else if (event.key === "ArrowLeft") {
				this.moveNote("left", noteId);
			} else if (event.key === "ArrowDown" || event.key === "ArrowUp") {
				event.preventDefault();
				const allNotes = [
					...this.uncategorizedNotes,
					...this.categories.flatMap((category) => category.notes),
				];
				const currentIndex = allNotes.findIndex((note) => note.id === noteId);
				const newIndex =
					event.key === "ArrowDown"
						? Math.min(currentIndex + 1, allNotes.length - 1)
						: Math.max(currentIndex - 1, 0);
				const newFocusedNote = allNotes[newIndex];
				const newFocusedElement = document.querySelector(
					`[data-note-id="${newFocusedNote.id}"]`
				);
				if (newFocusedElement) {
					newFocusedElement.focus();
				}
			}
		},

		findNoteById(id) {
			return (
				this.uncategorizedNotes.find((note) => note.id === id) ||
				this.categories
					.flatMap((category) => category.notes)
					.find((note) => note.id === id)
			);
		},

		moveNoteTo(note, targetCategoryName) {
			if (note.category !== targetCategoryName) {
				// Remove the note from its current category
				const currentCategory = this.categories.find(
					(category) => category.name.toLowerCase() === note.category
				);
				if (currentCategory) {
					currentCategory.notes = currentCategory.notes.filter(
						(n) => n.id !== note.id
					);
				} else {
					// If the note is uncategorized
					this.uncategorizedNotes = this.uncategorizedNotes.filter(
						(n) => n.id !== note.id
					);
				}

				// Add the note to the target category
				if (targetCategoryName === "uncategorized") {
					this.uncategorizedNotes.push(note);
				} else {
					const targetCategory = this.categories.find(
						(category) => category.name.toLowerCase() === targetCategoryName
					);
					if (targetCategory) {
						targetCategory.notes.push(note);
					}
				}

				// Update the note's category property
				note.category = targetCategoryName;
			}
		},

		async moveNote(direction, noteId) {
			if (!this.findNoteById(noteId).selected) {
				return;
			}
			let uncategorizedIndex = this.uncategorizedNotes.findIndex(
				(note) => note.id === noteId
			);
			let categoryIndex = -1;
			let category = null;

			if (uncategorizedIndex === -1) {
				this.categories.some((cat, catIndex) => {
					categoryIndex = cat.notes.findIndex((note) => note.id === noteId);
					if (categoryIndex > -1) {
						category = cat;
						return true;
					}
					return false;
				});
			}

			if (direction === "right" && uncategorizedIndex > -1) {
				const note = this.uncategorizedNotes.splice(uncategorizedIndex, 1)[0];
				this.categories[0].notes.push(note);
			} else if (direction === "left" && categoryIndex > -1) {
				const catIndex = this.categories.indexOf(category);
				if (catIndex === 0) {
					const note = category.notes.splice(categoryIndex, 1)[0];
					this.uncategorizedNotes.push(note);
				} else {
					const note = category.notes.splice(categoryIndex, 1)[0];
					this.categories[catIndex - 1].notes.push(note);
				}
			} else if (direction === "right" && categoryIndex > -1) {
				const catIndex = this.categories.indexOf(category);
				if (catIndex === this.categories.length - 1) {
					const note = category.notes.splice(categoryIndex, 1)[0];
					this.categories[0].notes.push(note);
				} else {
					const note = category.notes.splice(categoryIndex, 1)[0];
					this.categories[catIndex + 1].notes.push(note);
				}
			}

			// Wait for the next DOM update cycle before updating the focus
			await this.$nextTick();

			// Refocus the moved note
			const movedNoteElement = document.querySelector(
				`[data-note-id="${noteId}"]`
			);
			if (movedNoteElement) {
				movedNoteElement.focus();
			}
		},

		selectNote(note) {
			this.selectedNoteId = note.id;
			// Clear all other selected notes
			this.uncategorizedNotes.forEach((n) => (n.selected = false));
			this.categories.forEach((category) => {
				category.notes.forEach((n) => (n.selected = false));
			});

			// Toggle the clicked note
			note.selected = !note.selected;
		},

		dragStart(event, note) {
			this.selectNote(note);
			this.$refs.dnd.setAttribute("data-selected-note", "true");
			const originalCategory = this.categories.find((category) =>
				category.notes.some((n) => n.id === note.id)
			);
			const noteData = {
				...note,
				originalCategory: originalCategory ? originalCategory.name : null,
			};
			event.dataTransfer.setData("text/plain", JSON.stringify(noteData));
			event.target.classList.add("dragging");
			const categories = document.querySelectorAll(".category");
			categories.forEach((category) => {
				const landingPad = document.createElement("div");
				landingPad.classList.add("landing-pad");
				landingPad.textContent = "⇩";
				category.appendChild(landingPad);
			});
		},

		dragEnd(event) {
			event.dataTransfer.clearData();
			if (this.draggedNote) {
				this.$refs.dnd.removeChild(this.draggedNote);
				this.draggedNote = null;
			}
			event.target.classList.remove("dragging");
			const landingPads = document.querySelectorAll(".landing-pad");
			landingPads.forEach((landingPad) => {
				landingPad.remove();
			});
			this.$refs.dnd.removeAttribute("data-selected-note");
		},

		drop(event) {
			event.preventDefault();
			try {
				const noteData = JSON.parse(event.dataTransfer.getData("text/plain"));
				const targetElement = event.target;
				const categoryElement = targetElement.closest(".category");
				const uncategorizedElement = targetElement.closest(".uncategorized");

				// Find the note
				const note = this.findNoteById(noteData.id);

				let targetCategoryName = null;

				if (categoryElement) {
					targetCategoryName = categoryElement
						.getAttribute("data-category-name")
						.toLowerCase();
				} else if (uncategorizedElement) {
					targetCategoryName = "uncategorized";
				} else {
					console.log("Invalid drop target. The note was not moved.");
					return;
				}

				const originalCategoryName = noteData.originalCategory
					? noteData.originalCategory.toLowerCase()
					: "uncategorized";

				// Check if the note is being moved within the same category
				if (originalCategoryName === targetCategoryName) {
					console.log("The note was not moved as it is the same category.");
					return;
				}

				// Remove the note from the uncategorizedNotes or its original category
				if (originalCategoryName === "uncategorized") {
					this.uncategorizedNotes = this.uncategorizedNotes.filter(
						(n) => n.id !== noteData.id
					);
				} else {
					const originalCategory = this.categories.find(
						(c) => c.name.toLowerCase() === originalCategoryName
					);
					originalCategory.notes = originalCategory.notes.filter(
						(n) => n.id !== noteData.id
					);
				}

				// Make a deep copy of the note object before modifying it
				const newNote = { ...note };

				// Add the note to the target category or the uncategorizedNotes
				if (targetCategoryName === "uncategorized") {
					this.uncategorizedNotes.push(newNote);
				} else {
					const targetCategory = this.categories.find(
						(c) => c.name.toLowerCase() === targetCategoryName
					);
					targetCategory.notes.push(newNote);
				}
			} catch (error) {
				console.error("Error parsing JSON data:", error);
			}
		},

		dropNote(categoryName) {
			// Find the selected note
			const selectedNote = this.uncategorizedNotes.find(
				(note) => note.selected
			);
			if (!selectedNote) return;

			// Add the selected note to the category
			const category = this.categories.find(
				(c) => c.name.toLowerCase() === categoryName
			);
			if (!category) return;

			category.notes.push(selectedNote);
			selectedNote.selected = false;

			// Remove the note from the uncategorized area
			this.uncategorizedNotes = this.uncategorizedNotes.filter(
				(note) => note !== selectedNote
			);
			this.announceCategoryChange(selectedNote, categoryName);
		},

		handleDragOver(event) {
			event.preventDefault(); // Prevent default to allow dropping

			const scrollThreshold = 50; // Set a threshold for scrolling (in pixels)
			const viewportHeight = window.innerHeight;
			const mouseY = event.clientY;

			// If mouseY is within scrollThreshold pixels from the bottom, scroll down
			if (viewportHeight - mouseY < scrollThreshold) {
				window.scrollBy({ top: 10, behavior: "smooth" });
			}

			// If mouseY is within scrollThreshold pixels from the top, scroll up
			if (mouseY < scrollThreshold) {
				window.scrollBy({ top: -10, behavior: "smooth" });
			}
		},

		handleDragLeave(event) {
			event.preventDefault();
		},

		hoverCategory(event, isHover) {
			if (this.$refs.dnd.getAttribute("data-selected-note") === "true") {
				event.target
					.closest(".category")
					[isHover ? "classList" : "classList"].toggle("hover-with-selected");
			}
		},

		clickCategory(event) {
			if (this.$refs.dnd.getAttribute("data-selected-note") === "true") {
				const selectedNote = this.uncategorizedNotes.find(
					(note) => note.selected
				);
				if (selectedNote) {
					const categoryName =
						event.target.closest(".category").dataset.categoryName;
					const category = this.categories.find(
						(category) => category.name.toLowerCase() === categoryName
					);
					if (category) {
						this.moveNote(selectedNote, category);
					}
				}
			}
		},

		deselectAllNotes() {
			this.uncategorizedNotes.forEach((note) => {
				note.selected = false;
			});
			this.categories.forEach((category) => {
				category.notes.forEach((note) => {
					note.selected = false;
				});
			});
		},
		touchStart(event, note) {
			this.selectNote(note);
			this.draggedNote = note;
			event.target.classList.add("dragging");
			const categories = document.querySelectorAll(".category");
			categories.forEach((category) => {
				const landingPad = document.createElement("div");
				landingPad.classList.add("landing-pad");
				landingPad.textContent = "⇩";
				category.appendChild(landingPad);
			});
		},

		touchMove(event) {
			event.preventDefault();
			const touch = event.touches[0];
			const targetElement = document.elementFromPoint(
				touch.clientX,
				touch.clientY
			);

			if (targetElement && targetElement.closest(".category")) {
				this.overCategory = targetElement
					.closest(".category")
					.getAttribute("data-category-name");
			}
		},

		touchEnd(event) {
			event.target.classList.remove("dragging");
			const landingPads = document.querySelectorAll(".landing-pad");
			landingPads.forEach((landingPad) => {
				landingPad.remove();
			});
			if (this.draggedNote && this.overCategory) {
				this.moveNoteTo(this.draggedNote, this.overCategory);
			}
			this.draggedNote = null;
			this.overCategory = null;
		},
	},
	mounted() {
		document.addEventListener("dragover", this.handleDragOver);
		document.addEventListener("dragleave", this.handleDragLeave);
		this.$watch(
			() => document.activeElement,
			(newActiveElement) => {
				if (newActiveElement) {
					this.focusedNoteId = parseInt(
						newActiveElement.getAttribute("data-note-id"),
						10
					);
				}
			},
			{ flush: "post" }
		);
	},
	beforeUnmount() {
		document.removeEventListener("dragover", this.handleDragOver);
		document.removeEventListener("dragleave", this.handleDragLeave);
	},
};
</script>

<style scoped>
/* General styles */
.highlight {
	background-color: yellow;
}

ul li::before {
	content: unset;
}

.app {
	padding-right: 20px;
	padding-left: 20px;
	display: grid;
	grid-template-columns: 1fr 1fr;
	gap: 1rem;
	justify-content: space-evenly;
}

#app ul {
	display: flex;
	flex-wrap: wrap;
	list-style: none;
}

h2 {
	font-size: 1.5rem;
	font-weight: bold;
	margin-bottom: 1rem;
	text-align: center;
}

.landing-pad {
	position: absolute;
	left: 50%;
	transform: translate(-50%);
	border: 2px dashed #aaa;
	padding: 10px;
	margin: 10px 0;
	text-align: center;
	background-color: #f0f0f0;
	font-size: 36px;
	color: #aaa;
	user-drag: none;
	user-select: none;
}

/* Note styles */
.note {
	cursor: pointer;
	background-color: #feffe0;
	border: 1px solid #ddd;
	padding: 8px;
	padding-left: 20px;
	padding-right: 20px;
	font-size: 14px;
	line-height: 1.5;
	border-radius: 4px;
	min-width: 150px;
	max-width: 250px;
	flex: 1;
	margin-right: 20px;
	margin-bottom: 20px;
	box-shadow: 5px 5px 5px #888888;
	position: relative;
}

.note.selected {
	box-shadow: 0 0px 10px rgba(0, 0, 0, 0.2);
	transform: scale(1.05);
}

.note:focus-visible {
	outline: 2px solid #3f51b5;
}

.note.dragging {
	opacity: 0.1;
}

/* Container for all categories */
.categories {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
	gap: 16px;
}

.category:hover-with-selected {
	border-color: purple;
}

/* Container for categories and uncategorized notes */
.category,
.uncategorized {
	display: flex;
	flex-wrap: wrap;
	gap: 8px;
	padding: 8px;
	border: 1px solid #ccc;
	margin-bottom: 16px;
	align-content: flex-start;
}

/* Uncategorized area styles */
.uncategorized {
	background-color: #f0f0f0;
	border-radius: 0.5rem;
	min-width: 200px;
	padding: 1rem;
	grid-column: 1 / span 1;
}

/* Category area styles */
.category {
	position: relative;
	border: 2px solid #333;
	border-radius: 0.5rem;
	padding: 1rem;
	background-color: #fff;
}

.sr-only {
	border: 0;
	clip: rect(0, 0, 0, 0);
	height: 1px;
	margin: -1px;
	overflow: hidden;
	padding: 0;
	position: absolute;
	width: 1px;
}
</style>
