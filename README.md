
# Standar Operating Procedure 

## Content

- [Single responsibility principle](#single-responsibility-principle)

### **principle**

A class should have only one responsibility.

Bad:

```php
public function update(Request $request): string
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'events' => 'required|array:date,type'
    ]);

    foreach ($request->events as $event) {
        $date = $this->carbon->parse($event['date'])->toString();

        $this->logger->log('Update event ' . $date . ' :: ' . $);
    }

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

Good:

```php
public function update(UpdateRequest $request): string
{
    $this->logService->logEvents($request->events);

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

### **principle**

A class should have only one responsibility.

Bad:

```php
public function update(Request $request): string
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'events' => 'required|array:date,type'
    ]);

    foreach ($request->events as $event) {
        $date = $this->carbon->parse($event['date'])->toString();

        $this->logger->log('Update event ' . $date . ' :: ' . $);
    }

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

Good:

```php
public function update(UpdateRequest $request): string
{
    $this->logService->logEvents($request->events);

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

### **principle**

A class should have only one responsibility.

Bad:

```php
public function update(Request $request): string
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'events' => 'required|array:date,type'
    ]);

    foreach ($request->events as $event) {
        $date = $this->carbon->parse($event['date'])->toString();

        $this->logger->log('Update event ' . $date . ' :: ' . $);
    }

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

Good:

```php
public function update(UpdateRequest $request): string
{
    $this->logService->logEvents($request->events);

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

### **Single responsibility principle**

A class should have only one responsibility.

Bad:

```php
public function update(Request $request): string
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'events' => 'required|array:date,type'
    ]);

    foreach ($request->events as $event) {
        $date = $this->carbon->parse($event['date'])->toString();

        $this->logger->log('Update event ' . $date . ' :: ' . $);
    }

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```

Good:

```php
public function update(UpdateRequest $request): string
{
    $this->logService->logEvents($request->events);

    $this->event->updateGeneralEvent($request->validated());

    return back();
}
```
