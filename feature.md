The original ZDR was: "dynamic storage zones" ZDP 100032570. This ZDR has undergone significant adaptations for WM. Most of these changes have been implemented and proven effective. However, during the last implementation, WM had some complaints about certain functionalities. A meeting between @Eric ZenzMatzl, @Michael LANG, and @Dominik FUCHSBICHLER resulted in the following conclusions:

=============

**New feature branch:** `feature/zdp-100032570` based on `feature/zdp-100032570.dynamic-storage-zones`.  
**Create merge request:** to `feature/zdp-100032570.dynamic-storage-zones` when finished.

**Softening checks in the database:**

The shrunk zone must have more "empty container positions" and "free positions" available than the containers in the shrunk zone that will be moved to the expanded zone.

**DB => 3 MD**

**Additional check when executing the reassignment:**

In addition to the forecast check to see if the reassignment is feasible, there should also be a check during the execution of the reassignment to ensure it is possible.

**DB => 2 MD**

**Additional safety check:**

The database must ensure that at least one RackLocation per Zone/Level/Aisle remains free, regardless of the zone size.

**DB => 5 MD**

**Automatic reassignment in Spider:**

When a container with a product is relocated from the expanded zone to the shrunk zone and there is no space, but there are enough empty containers that can be moved to the expanded zone, Spider will automatically create a relocation of an empty container to the expanded zone.

**Spider => 3 MD**

**In OverlappingZonesManager:**

```java
/**
 * Selects a container for the given allocation order, based on the
 * container class and source zone of the order.
 *
 * @param allocationOrder
 * @return a container for the class which is currently assigned to the
 * source zone of the order.
 */
private Container selectContainerForReallocationOrder(AllocationOrder allocationOrder) {
    // Implementation here
}
```

We shouldn't limit the number of relocations, as they are already implicitly limited. However, we need to ensure that we prefer empty containers over non-empty containers. Introduce a new `Container` member `empty` in the same manner as `classId`, `length`, and `weight` attributes. Use this attribute in `OverlappingZonesManager` to prefer empty containers over non-empty containers. That should be all there is to it.
