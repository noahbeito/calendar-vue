# calendarVue

Calendar            |  Event Modal
:-------------------------:|:-------------------------:
![image](https://github.com/noahbeito/calendar-vue/assets/90667844/779ea728-ea1f-4893-a8fa-f7396391dd17)  |  ![image](https://github.com/noahbeito/calendar-vue/assets/90667844/e2459aae-5ced-4107-8d02-dffe24342d2f)

## Notes
- I decided to keep everything in a single file component to keep the calendar and its functionaty as modular as possible. It might make sense in the scope of a larger project to make the modal its own reusable component, especially if there were other locations where a modal would be needed.

- Additional add ons:
    - Today button: this returns the calendar to the current month of today's date
    - Add and view events: this allows the user to click on a day and a modal will pop up. The modal will show any existing events scheduled for that day and allow the user to add new events. Events will be displayed in chronological order. When the modal is closed, days that have events scheduled will show a small blue dot under the day number.
    - Events do not persist if you reload the page as I didn't setup any methods of data persistence for the scope of this project.
- Potential Improvements:
    - There are a couple console errors that I did not yet address. This is something I would ideally like to correct.
    - Event editing and deletion
    - make more modular by being able to pass props that can customize the size and alignment of the calendar. This would likely make it easier to add into an existing application.
    - Add data persistence via a database or local storage.
    - Add accessibility i.e. ARIA labels for screen readers.
    - add testing, unit and end-to-end tests



## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```