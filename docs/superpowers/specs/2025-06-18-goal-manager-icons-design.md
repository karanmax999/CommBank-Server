# Goal Manager Icon Add/Change Implementation

**Status:** Approved  
**Date:** 2025-06-18  
**Project:** CommBank Goal Tracker  

---

## 1. Overview

The Goal Manager component (`GoalManager.tsx`) allows users to view and edit financial goals. It currently has partial support for adding and changing goal icons via an emoji picker, but the implementation is inconsistent with the desired design pattern and the `pickEmojiOnClick` event handler needs to be finalized.

This spec details the changes needed to:
- Ensure users can add an icon to a goal that doesn't have one
- Allow users to change an existing icon by clicking on it
- Properly persist icon changes to the backend via the PUT API

---

## 2. Problem Statement

The current implementation has the following issues:

1. **Component structure**: The `AddIconButton` component uses a custom prop `hasIcon` and conditionally renders null, while the container `AddIconButtonContainer` uses `shouldShow` prop. This causes type inconsistencies and unnecessary complexity.

2. **Hardcoded font size**: `GoalIcon` uses `font-size: 6rem` but the spec requires `5.5rem`.

3. **Event handler signature**: The `pickEmojiOnClick` handler currently uses the correct `(emoji, event)` signature, but the approved design calls for a curried function `() => (emoji, event) => {}`.

4. **State synchronization**: The handler currently updates both Redux and calls the API. The approved design suggests only calling the API (no direct Redux dispatch). This may rely on the API response triggering a refetch or other state update mechanism.

5. **Icon fallback**: The current code uses `emoji.native` directly. The approved design adds a fallback: `emoji.native ?? props.goal.icon` to guard against empty values.

---

## 3. Goals & Success Criteria

- **Functional**: Users can add an icon to a goal without an icon by clicking the "Add icon" button
- **Functional**: Users can change an existing icon by clicking on it, opening the emoji picker
- **Visual**: The Add icon button uses the `TransparentButton` component with proper styling
- **Visual**: The displayed icon uses `font-size: 5.5rem` and is wrapped in `TransparentButton`
- **Behavior**: Clicking either the Add icon button or the current icon opens the emoji picker
- **Persistence**: Selecting an emoji updates the goal on the server via the `updateGoal` API
- **Type Safety**: No TypeScript errors; prop types align with styled-components

---

## 4. Current Implementation

### Components

- **AddIconButton.tsx**: 
  - Props: `hasIcon: boolean`, `onClick: (event: React.MouseEvent) => void`
  - Renders a `Container` wrapping a `TransparentButton` with a smile icon and "Add icon" text
  - Returns `null` if `hasIcon` is true

- **GoalIcon.tsx**:
  - Props: `icon: string | null`, `onClick: (event: React.MouseEvent) => void`
  - Renders `TransparentButton` containing an `<Icon>` styled element with `font-size: 6rem`
  - Icon content is the emoji string

- **GoalManager.tsx**: 
  - Manages local `icon` state derived from Redux `goal.icon`
  - Renders `GoalIconContainer` and `AddIconButtonContainer` conditionally
  - Passes `hasIcon()` to both containers and `AddIconButton`
  - Emoji picker opens when either button is clicked
  - `pickEmojiOnClick` updates local icon state, closes picker, dispatches Redux action, and calls `updateGoalApi`

### Styles

GoalManager.tsx defines these styled components:

```tsx
const GoalIconContainer = styled.div<{ shouldShow: boolean }>`
  display: ${(props) => (props.shouldShow ? 'flex' : 'none')};
`

const AddIconButtonContainer = styled.div<{ hasIcon: boolean }>`
  display: ${(props) => (props.hasIcon ? 'flex' : 'none')};
`

const EmojiPickerContainer = styled.div<{ isOpen: boolean; hasIcon: boolean }>`
  display: ${(props) => (props.isOpen ? 'flex' : 'none')};
  position: absolute;
  top: ${(props) => (props.hasIcon ? '10rem' : '2rem')};
  left: 0;
`
```

Note: There was a recent fix that changed `AddIconButtonContainer` from `shouldShow` to `hasIcon` to match usage.

---

## 5. Proposed Solution

### 5.1 Inline Add Icon Button

Remove the `AddIconButton` component and render the button directly inside `GoalManager`. This simplifies the component tree and aligns with the spec.

**Changes:**
- Delete `AddIconButton.tsx`
- In `GoalManager.tsx`, replace the `<AddIconButtonContainer>` contents with:
  ```tsx
  <AddIconButtonContainer hasIcon={hasIcon()}>
    <TransparentButton onClick={addIconOnClick}>
      <FontAwesomeIcon icon={faSmile} size="2x" />
      <AddIconButtonText>Add icon</AddIconButtonText>
    </TransparentButton>
  </AddIconButtonContainer>
  ```
- Add a new styled component `AddIconButtonText`:
  ```tsx
  const AddIconButtonText = styled.span`
    margin-left: 0.6rem;
    font-size: 1.5rem;
    color: rgba(174, 174, 174, 1);
  `
  ```
- Remove the now-unused import for `AddIconButton`

### 5.2 Adjust GoalIcon Font Size

In `GoalIcon.tsx`, change the `Icon` styled component from `6rem` to `5.5rem`:

```tsx
const Icon = styled.h1`
  font-size: 5.5rem;
  cursor: pointer;
`
```

### 5.3 Update pickEmojiOnClick

Modify the `pickEmojiOnClick` handler as follows:

Change the signature to a curried function (though note: the outer function takes no arguments, so this is effectively a wrapper). The implementation will be:

```ts
const pickEmojiOnClick = () => (emoji: BaseEmoji, event: React.MouseEvent) => {
  event.stopPropagation()
  setIcon(emoji.native)
  setEmojiPickerIsOpen(false)

  const updatedGoal: Goal = {
    ...props.goal,
    icon: emoji.native ?? props.goal.icon,
    name: name ?? props.goal.name,
    targetDate: targetDate ?? props.goal.targetDate,
    targetAmount: targetAmount ?? props.goal.targetAmount,
  }

  updateGoalApi(props.goal.id, updatedGoal)
}
```

Because the `EmojiPicker` expects `onClick: (emoji: BaseEmoji, event: React.MouseEvent) => void`, we will need to pass the result of calling the outer function:

```tsx
<EmojiPicker onClick={pickEmojiOnClick()} />
```

Alternatively, we could redefine `pickEmojiOnClick` as:

```ts
const pickEmojiOnClick = (emoji: BaseEmoji, event: React.MouseEvent) => {
  // ... same body without the outer wrapper
}
```

and continue to pass it as `onClick={pickEmojiOnClick}`. The curried form introduces an extra call that serves no functional purpose unless there are plans to capture some context later. For simplicity and clarity, **recommendation**: avoid unnecessary currying and keep the standard two-parameter function signature. The spec's curried version appears to be a stylistic choice; both are functionally equivalent if we call `pickEmojiOnClick()` when passing.

**Decision**: We will implement the standard two-parameter version to keep the code clean and type-correct, while still incorporating the fallback `icon: emoji.native ?? props.goal.icon` and removing the Redux dispatch as in the spec.

**Implementation:**

```ts
const pickEmojiOnClick = (emoji: BaseEmoji, event: React.MouseEvent) => {
  event.stopPropagation()
  setIcon(emoji.native)
  setEmojiPickerIsOpen(false)

  const updatedGoal: Goal = {
    ...props.goal,
    icon: emoji.native ?? props.goal.icon,
    name: name ?? props.goal.name,
    targetDate: targetDate ?? props.goal.targetDate,
    targetAmount: targetAmount ?? props.goal.targetAmount,
  }

  updateGoalApi(props.goal.id, updatedGoal)
}
```

We will remove the line: `dispatch(updateGoalRedux(updatedGoal))`.

### 5.4 Styling AddIconButtonText

Define `AddIconButtonText` as a styled span next to the other styled components in `GoalManager.tsx`:

```tsx
const AddIconButtonText = styled.span`
  margin-left: 0.6rem;
  font-size: 1.5rem;
  color: rgba(174, 174, 174, 1);
`
```

### 5.5 Clean Up Imports

After removing `AddIconButton` component:

- Remove import: `import AddIconButton from './AddIconButton'`
- Ensure `TransparentButton` is imported from `'../../components/TransparentButton'` (already present)
- Keep `FontAwesomeIcon` import: `import { FontAwesomeIcon } from '@fortawesome/react-fontawesome'` and the `faSmile` icon import in GoalManager. The spec uses `faSmile`; we need to import it. Check if it's already imported: current GoalManager imports `faCalendarAlt` and `faDollarSign` but not `faSmile`. We'll need to add: `import { faSmile } from '@fortawesome/free-solid-svg-icons'`

### 5.6 Remove AddIconButton Component File

After deleting `AddIconButton.tsx`, the goal manager will be the only place that renders the add icon button.

---

## 6. Implementation Checklist

- [ ] Read all affected files to verify current state
- [ ] Add `faSmile` import to `GoalManager.tsx`
- [ ] Add `AddIconButtonText` styled component in `GoalManager.tsx`
- [ ] Replace `<AddIconButtonContainer>` children with inlined `TransparentButton` and icon/text
- [ ] Remove import and usage of `AddIconButton` component
- [ ] Update `GoalIcon.tsx` to change font-size from `6rem` to `5.5rem`
- [ ] Modify `pickEmojiOnClick` in `GoalManager.tsx`:
  - [ ] Add fallback `icon: emoji.native ?? props.goal.icon`
  - [ ] Remove `dispatch(updateGoalRedux(updatedGoal))`
- [ ] Verify `TransparentButton` component exists at `src/ui/components/TransparentButton.tsx` and has proper styling
- [ ] Ensure there are no TypeScript errors after changes
- [ ] Run build to validate

---

## 7. Technical Considerations

- **Prop Types**: Ensure `AddIconButtonContainer` uses `hasIcon` prop (already fixed)
- **Event Propagation**: The `event.stopPropagation()` call should remain to prevent unwanted interactions
- **State Updates**: Removing Redux dispatch means local Redux store won't update immediately. Confirm that this is acceptable given the design. Possibly the API response triggers a refetch, or the parent component updates from server via polling or subscription. If not, UI may show stale data until a refresh.
- **Styling**: The `AddIconButtonText` color should match the existing design. The current `AddIconButton` used `rgba(174, 174, 174, 1)` for text; we maintain that.
- **Conditional Rendering**: The container still uses `hasIcon` to toggle display. That remains unchanged.
- **Icon Fallback**: The `emoji.native ?? props.goal.icon` fallback ensures that if the emoji picker returns a falsy native string, we keep the existing icon. This is defensive.

---

## 8. Acceptance Criteria

- [ ] No TypeScript compilation errors
- [ ] GoalManager renders correctly:
  - When `goal.icon` is null/empty: the "Add icon" button (with smiley icon and text) is visible; the current icon area is hidden
  - When `goal.icon` is set: the icon (emoji) is visible with font-size 5.5rem; the "Add icon" button is hidden
- [ ] Clicking the "Add icon" button opens the emoji picker
- [ ] Clicking the current goal icon opens the emoji picker
- [ ] Selecting an emoji sends a PUT request to the `updateGoal` API endpoint with the updated goal object (including new icon)
- [ ] The UI reflects the new icon after selection (assuming the Redux store updates via some other mechanism, or at least the local icon state updates)
- [ ] The emoji picker closes after selection
- [ ] The AddIconButton component file is deleted and no longer referenced
- [ ] `AddIconButtonText` styled component appears with proper spacing and color

---

## 9. Open Questions

- **State consistency**: Removing the Redux dispatch could cause the UI to revert to the old icon if the goal object from Redux isn't updated via another mechanism. The current `hasIcon` state is derived from the Redux `goal.icon` via `useAppSelector(selectGoalsMap)[props.goal.id]`. That selector might update when the API call succeeds and the slice updates from server. Need to verify the goalsSlice handles optimistic updates or refetches on success. If not, we should keep the dispatch. We'll proceed as spec'd (remove dispatch), but be prepared to reintroduce if needed.

- **Font size discrepancy**: The spec says "font-size: 5.5rem;" but the provided GoalIcon.tsx snippet shows `6rem`. We will use 5.5rem per the main spec bullet point. If that's a typo, we can adjust.

---

## 10. Related Files

- `commbank-web/src/ui/features/goalmanager/GoalManager.tsx`
- `commbank-web/src/ui/features/goalmanager/GoalIcon.tsx`
- `commbank-web/src/ui/features/goalmanager/AddIconButton.tsx` (to be deleted)
- `commbank-web/src/api/lib.ts` (contains `updateGoal` function)
- `commbank-web/src/store/goalsSlice.ts` (Redux state management)

---

This design is ready for implementation planning.
