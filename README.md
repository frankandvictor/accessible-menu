# Accessible Menu

> A JavaScript class which attaches navigation menu behavior to markup.

## Features

- **Fully accessible** - Incorporates all essential practices for accessibility.
- **Style-agnostic** - Makes very few style assumptions and mandates.
- **Highly flexible** - Allows for infinitely-nested menus.

## Terminology

- **Menu** - An element which contains **items**.
- **Parent Menu** - A **menu** which has at least one **child menu**.
- **Child Menu** - A **menu** which has a parent menu (also known as a "submenu").
- **Root Menu** - A **menu** which has no **parent menu**.
- **Item** - An element within a **menu** which contains a **button**, and optionally, a **child menu**.
- **Button** - An element within an **item** which triggers an action such as navigating to a link or opening a **child menu**.
- **Menu Button** - A **button** which toggles the visibility of a **root menu**.

## Structure

- The **root menu** must exist.
- The **root menu** must contain at least one **items**.
- A **menu button** may exist.
- An **item** must contain one **button**.
- An **item** may contain one **child menu**.

This pattern is infinitely nestable. See the example menu structure below:

```
menu button

menu
├───item
│   └───button
├───item
│   ├───button
│   └───menu
│       ├───item
│       │   └───button
│       └───item
│           └───button
├───item
│   ├───button
│   └───menu
│       ├───item
│       │   └───button
│       ├───item
│       │   ├───button
│       │   └───menu
│       │       ├───item
│       │       │   └───button
│       │       ├───item
│       │       │   └───button
│       │       └───item
│       │           └───button
│       └───item
│           └───button
├───item
│   └───button
└───item
    └───button
```

## Accessibility

### Attributes

#### Menu Button

- `id` is set to a randomly-generated string (if no `id` is found).
- `aria-label` is set to `Menu Button` (if no `aria-label` or `innerText` is found).
- `role` is set to `button`.
- `aria-haspopup` is to to `true`.
- `aria-controls` is set to its menu's `id`.
- `aria-expanded` is set to `false`.
- `aria-expanded` is dynamically updated as part of a `expand/collapse` pattern.

#### Menu

- `aria-label` is set to `Menu` (if no `aria-label` is found).
- `id` is set to a randomly-generated string (if no `id` is found).
- `role` is set to `menubar` (or `menu` if it is toggleable via a menu button).

#### Item

- No attribute modifications.

#### Button

- `id` is set to a randomly-generated string.
- `tabindex` is set to `-1` (or `0` if it is the first button in the root menu).
- `tabindex` is dynamically updated as part of a `roving tabindex` pattern.
- `role` is set to `menuitem`.
- If the item has a menu...
	- `aria-haspopup` is set to `true`.
	- `aria-controls` is set to its menu's `id`.
	- `aria-expanded` is set to `false`.
	- `aria-expanded` is dynamically updated as part of a `expand/collapse` pattern.

### Mouse Control

- Clicking a button for an item without a menu will gain no special behavior.
- Clicking a button for an item with a menu will toggle the visibility of that menu.
- Clicking outside of an open menu (except the root menu) will cause the menu to close, along with any open child menus.

### Keyboard Control

When focus is on a button:

- **Space**
	- If the item has a menu...
		- If the menu is closed...
			- Opens the menu and moves focus to the first button in the menu.
		- If the menu is open...
			- Closes the menu and any open child menus.
- **Enter**
	- If the item has a menu...
		- If the menu is closed...
			- Opens the menu and moves focus to the first button in the menu.
		- If the menu is open...
			- Closes the menu and any open child menus.
- **Escape**
	- If the item is in a non-root menu...
		- If the menu is open...
			- Closes the menu and any open child menus. Moves focus to the button which opened it.
- **Tab**
	- Closes all menus. Moves focus to the next focusable element outside the menu system.
- **Shift + Tab**
	- Closes all menus. Moves focus to the previous focusable element outside the menu system.
- **Left Arrow** - Moves focus to the previous button in the menu. If focus is on the first button, moves focus to the last button.
- **Right Arrow** - Moves focus to the next button in the menu. If focus is on the last button, moves focus to the first button.
- **Up Arrow** - Moves focus to the previous button in the menu. If focus is on the first button, moves focus to the last button.
- **Down Arrow** - Moves focus to the next button in the menu. If focus is on the last button, moves focus to the first button.
- **Home** - Moves focus to the first button in the menu.
- **End** - Moves focus to the last button in the menu.
- **Page Up** - Moves focus to the first button in the menu.
- **Page Down** - Moves focus to the last button in the menu.
- **Character** - Moves focus to next button in the menu that starts with the typed character (wrapping around to the beginning, if necessary). If none of the buttons start with the typed character, focus does not move.

## Usage

The menu markup should be written such that **menu** elements have the class `.menu`, **item** elements have the class `.item`, and **button** elements have the class `.button`.

To attach behavior to the menu, simply instantiate a new `Menu` with the root menu as the only argument. The `Menu` class will recursively construct instances for any child menus.

``` html
<div class="menu">
	<div class="item">
		<a class="button" href="#">Lorem Ipsum</a>
	</div>

	<div class="item">
		<button class="button">Dolor Sit</button>

		<div class="menu">
			<div class="item">
				<a class="button" href="#">Amet Consectetur</a>
			</div>

			<!-- More items... -->
		</div>
	</div>

	<!-- More items... -->
</div>

<script>
	new Menu(document.querySelector('.menu'));
</script>
```

The code above will yield a visually persistent menu. If instead the menu should be toggleable (e.g. a mobile menu toggled via a hamburger button), pass in an external button as the second argument.

``` html
<button class="menu-button"></button>

<div class="menu">
	<!-- Items... -->
</div>

<script>
	new Menu(document.querySelector('.menu'), document.querySelector('.menu-button'));
</script>
```

### Transitions

By default, menus will open and close instantly. However, there are two built-in transition types to choose from, `fade` and `slide`. To set transition types, you'll need to pass in a third parameter into the class constructor. Because this menu system supports infinite nesting of child menus, this parameter is an array, where the index in the array corresponds to the depth of the target menu. For example, to create a menu system in which the root menu (toggled by a menu button) fades, while the child menus slide, this is the syntax:

``` js
new Menu(menu, menuButton, ['fade', 'slide']);
```

Transition settings are passed down from parent menu to child menu, so if the menu in the above example has tertiary and quaternary menus, they would slide. Because of this inheritence, if you wanted all menus to slide, you could simply pass in `['slide']`.

Note: In menu systems that do not have a menu button (i.e. visually persistent menus not toggled by a menu button), the transition settings for the root menu are still configured at index 0 of the array, even though this menu will never transition. For example, the code below indicates a menu without a menu button whose child menus fade and grandchild menus slide.

``` js
new Menu(menu, null, [null, 'fade', 'slide']);
```

Custom transitions are also supported. To use a custom transition, simply pass in an object that contains open and close functions rather than `'slide'` or `'fade'`. Each function is required to run the provided callback function when the animation is complete.

``` js
const customTransition = {
	open: (menu, callback) => {
		// Transition code...
	},
	close: (menu, callback) => {
		// Transition code...
	},
};

new Menu(menu, menuButton, [customTransition]);
```

### Events

If you need to run certain functions based on menu behavior, you can listen for events on the menu element. When the menu is opens, it dispatches `menuopen`. When the menu closes, it dispatches `menuclose`. See the example below:

``` js
const menu = document.querySelector('.menu');
new Menu(menu);
menu.addEventListener('menuopen', () => { console.log('Open'); });
menu.addEventListener('menuclose', () => { console.log('Close'); });
```

### Examples

[See full examples &raquo;](./examples)
