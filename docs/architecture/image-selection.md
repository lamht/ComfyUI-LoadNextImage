# Image selection and index behavior

This page explains how the loader behaves when image files or their timestamps change. The user-facing setup is in the [README](../../README.md).

## Ordering and saved position

Each execution scans the selected directory for supported image files and sorts them by ascending last-modified time. Files with the same timestamp are ordered by natural filename. The scan is repeated for every execution, so additions, removals, and timestamp changes affect the current list immediately.

`index.txt` stores a zero-based position in that ordered list, not an image identity. The loader reads it while holding the directory's index lock. Missing, invalid, negative, or out-of-range values start at position `0`. After a successful image load, the loader saves the position after the selected image, wrapping to `0` at the end.

Because the saved value is a position, changes to the list can change which image that position refers to. Adding or removing an image before it, or changing timestamps so files move around, can cause the next execution to repeat or skip an image. If an exact next-image identity must survive folder changes, the sequence needs to persist an image identity rather than only a position.

## Manual selection and preview

Choosing a filename loads that image once and saves the position after it as the next index. Previewing a selection does not read or modify `index.txt`.

The browser refreshes its filename choices when the node is initialized or configured, when its directory changes, and after execution. It does not watch the directory continuously. External file changes may therefore leave the open dropdown stale until one of those refreshes; a stale selection can fail if its file was removed.

## Failure behavior

An empty directory or an image that cannot be decoded causes execution to fail without advancing the index. If the image loads but saving the new index fails, the image is still returned with a warning; the next execution may select it again because the saved position did not advance.
