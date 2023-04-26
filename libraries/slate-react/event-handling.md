# Slate React Event Handling

默认情况下，`Editable` 组件带有一组事件处理程序，用于处理典型的富文本编辑行为（例如实现了自己的 `onCopy`、`onPaste`、`onDrop`、 `onKeyDown` 处理程序。

在某些情况下会想要扩展或者重写 Slate 的默认行为，可以通过将自定义的事件处理程序传递给 `Editable` 组件来实现。

Your custom event handler can control whether or not Slate should execute its own event handling for a given event after your handler runs depending on the return value of your event handler as described below.

```jsx
import {Editable} from 'slate-react';

function MyEditor() {
  const onClick = event => {
    // 实现自定义事件逻辑。。。

    // When no value is returned, Slate will execute its own event handler when
    // neither isDefaultPrevented nor isPropagationStopped was set on the event
  };

  const onDrop = event => {
    // 实现自定义事件逻辑。。。

    // No matter the state of the event, treat it as being handled by returning
    // true here, Slate will skip its own event handler
    return true;
  };

  const onDragStart = event => {
    // 实现自定义事件逻辑。。。

    // No matter the status of the event, treat event as *not* being handled by
    // returning false, Slate will execute its own event handler afterward
    return false;
  };

  return (
    <Editable
      onClick={onClick}
      onDrop={onDrop}
      onDragStart={onDragStart}
      {/*...*/}
    />
  )
}
```
