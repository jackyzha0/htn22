# Step by Step

## 1) Setup our own Cursor

```javascript
// setup our data
let me = {
  id: nanoid(),
  color: randomcolor(),
  x: 0,
  y: 0,
};

// initialize our little cursor :))
const cursor = cursorFactory(me.id, me.color);
const div = document.getElementById("cursor-layer");
div.appendChild(cursor);
document.onmousemove = (evt) => {
  // attach a listener for mouse movement
  if (me.x !== evt.pageX || me.y !== evt.pageY) {
    sendUpdate = true;
    me.x = evt.pageX;
    me.y = evt.pageY;
    cursor.style.setProperty("transform", `translate(${me.x}px, ${me.y}px)`);
    console.log(me)
  }
};
```

If we look at console now, we can see our data being updated as we move our cursor!

## 2) Let's replicate it!

```javascript
const doc = new Y.Doc();
const provider = new WebrtcProvider(
  "cursor-party",
  doc
);

// only send updates when we need to!
let replicated_cursors = doc.getMap('state');
setInterval(() => {
  replicated_cursors.set(me.id, me);
}, 80);

// make sure we delete our own mess when we exit
addEventListener('beforeunload', () => {
  replicated_cursors.delete(me.id);
});
```

## 3) Listen for other cursors


```javascript
replicated_cursors.observe(evt => {
  const updated_cursors = evt.changes.keys;
  updated_cursors.forEach((change, cursor_id) => {
    if (cursor_id !== me.id) {
      switch (change.action) {
        case 'add':
          const new_cursor = replicated_cursors.get(cursor_id);
          const new_cursor_div = cursorFactory(new_cursor.id, new_cursor.color);
          div.appendChild(new_cursor_div);
          new_cursor_div.style.setProperty("transform", `translate(${new_cursor.x}px, ${new_cursor.y}px)`)
          break;
        case 'update':
          const updated_cursor = replicated_cursors.get(cursor_id);
          const updated_cursor_div = document.getElementById(`cursor_${cursor_id}`);
          updated_cursor_div.style.setProperty("transform", `translate(${new_cursor.x}px, ${new_cursor.y}px)`)
        case 'delete':
          const old_cursor_div = document.getElementById(`cursor_${cursor_id}`);
          old_cursor_div.remove();
          break;
      }
    }
  })
})
```

## 4) Interpolate cursor positions

```javascript
const cursor_interp = new Map();

...
switch (change.action) {
  case 'add':
    // make a new cursor
    const new_cursor = replicated_cursors.get(cursor_id);
    const new_cursor_div = cursorFactory(new_cursor.id, new_cursor.color);
    div.appendChild(new_cursor_div);
    const add_point_closure = ([x, y]) => new_cursor_div.style.setProperty("transform", `translate(${x}px, ${y}px)`);
    const perfect_cursor = new PerfectCursor(add_point_closure);
    perfect_cursor.addPoint([new_cursor.x, new_cursor.y]);
    cursor_interp.set(cursor_id, perfect_cursor);      
    break;
  case 'update':
    const updated_cursor = replicated_cursors.get(cursor_id);
    cursor_interp.get(cursor_id).addPoint([updated_cursor.x, updated_cursor.y]);
    break;
  case 'delete':
    const old_cursor_div = document.getElementById(`cursor_${cursor_id}`);
    old_cursor_div.remove();
    cursor_interp.delete(cursor_id);
    break;
}
```
