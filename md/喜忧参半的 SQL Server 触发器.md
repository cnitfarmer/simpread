> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/hhhnicvscs/p/14444984.html)

SQL Server 触发器在非常有争议的主题。它们能以较低的成本提供便利，但经常被开发人员、DBA 误用，导致性能瓶颈或维护性挑战。

本文简要回顾了触发器，并深入讨论了如何有效地使用触发器，以及何时触发器会使开发人员陷入难以逃脱的困境。

虽然本文中的所有演示都是在 SQL Server 中进行的，但这里提供的建议是大多数数据库通用的。触发器带来的挑战在 MySQL、PostgreSQL、MongoDB 和许多其他应用中也可以看到。

**什么是触发器**

可以在数据库或表上定义 SQL Server 触发器，它允许代码在发生特定操作时自动执行。本文主要关注表上的 DML 触发器，因为它们往往被过度使用。相反，数据库的 DDL 触发器通常更集中，对性能的危害更小。

触发器是对表中数据更改时进行计算的一组代码。触发器可以定义为在插入、更新、删除或这些操作的任何组合上执行。MERGE 操作可以触发语句中每个操作的触发器。

触发器可以定义为 INSTEAD OF 或 AFTER。AFTER 触发器发生在数据写入表之后，是一组独立的操作，和写入表的操作在同一事务执行，但在写入发生之后执行。如果触发器失败，原始操作也会失败。INSTEAD OF 触发器替换调用的写操作。插入、更新或删除操作永远不会发生，而是执行触发器的内容。

触发器允许在发生写操作时执行 TSQL，而不管这些写操作的来源是什么。它们通常用于在希望确保执行写操作时运行关键操作，如日志记录、验证或其他 DML。这很方便，写操作可以来自 API、应用程序代码、发布脚本，或者内部流程，触发器无论如何都会触发。

**触发器是什么样的**

用 WideWorldImporters 示例数据库中的 Sales.Orders 表举例，假设需要记录该表上的所有更新或删除操作，以及有关更改发生的一些细节。这个操作可以通过修改代码来完成，但是这样做需要对表的代码写入中的每个位置进行更改。通过触发器解决这一问题，可以采取以下步骤:

1. 创建一个日志表来接受写入的数据。下面的 TSQL 创建了一个简单日志表，以及一些添加的数据点：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TABLE Sales.Orders_log
( Orders_log_ID int NOT NULL IDENTITY(1,1) 
 CONSTRAINT PK_Sales_Orders_log PRIMARY KEY CLUSTERED,
 OrderID int NOT NULL,
 CustomerID_Old int NOT NULL,
 CustomerID_New int NOT NULL,
 SalespersonPersonID_Old int NOT NULL,
 SalespersonPersonID_New int NOT NULL,
 PickedByPersonID_Old int NULL,
 PickedByPersonID_New int NULL,
 ContactPersonID_Old int NOT NULL,
 ContactPersonID_New int NOT NULL,
 BackorderOrderID_Old int NULL,
 BackorderOrderID_New int NULL,
 OrderDate_Old date NOT NULL,
 OrderDate_New date NOT NULL,
 ExpectedDeliveryDate_Old date NOT NULL,
 ExpectedDeliveryDate_New date NOT NULL,
 CustomerPurchaseOrderNumber_Old nvarchar(20) NULL,
 CustomerPurchaseOrderNumber_New nvarchar(20) NULL,
 IsUndersupplyBackordered_Old bit NOT NULL,
 IsUndersupplyBackordered_New bit NOT NULL,
 Comments_Old nvarchar(max) NULL,
 Comments_New nvarchar(max) NULL,
 DeliveryInstructions_Old nvarchar(max) NULL,
 DeliveryInstructions_New nvarchar(max) NULL,
 InternalComments_Old nvarchar(max) NULL,
 InternalComments_New nvarchar(max) NULL,
 PickingCompletedWhen_Old datetime2(7) NULL,
 PickingCompletedWhen_New datetime2(7) NULL,
 LastEditedBy_Old int NOT NULL,
 LastEditedBy_New int NOT NULL,
 LastEditedWhen_Old datetime2(7) NOT NULL,
 LastEditedWhen_New datetime2(7) NOT NULL,
 ActionType VARCHAR(6) NOT NULL,
 ActionTime DATETIME2(3) NOT NULL,
UserName VARCHAR(128) NULL);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

该表记录所有列的旧值和新值。这是非常全面的，我们可以简单地记录旧版本的行，并能够通过将新版本和旧版本合并在一起来了解更改的过程。最后 3 列是新增的，提供了有关执行的操作类型 (插入、更新或删除)、时间和操作人。

2. 创建一个触发器来记录表的更改:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TRIGGER TR_Sales_Orders_Audit
 ON Sales.Orders
 AFTER INSERT, UPDATE, DELETE
AS
BEGIN
 SET NOCOUNT ON;
 INSERT INTO Sales.Orders_log
 (OrderID, CustomerID_Old, CustomerID_New, 
 SalespersonPersonID_Old, SalespersonPersonID_New, 
 PickedByPersonID_Old, PickedByPersonID_New,
 ContactPersonID_Old, ContactPersonID_New, 
 BackorderOrderID_Old, BackorderOrderID_New, 
 OrderDate_Old, OrderDate_New, ExpectedDeliveryDate_Old,
 ExpectedDeliveryDate_New, 
 CustomerPurchaseOrderNumber_Old, 
 CustomerPurchaseOrderNumber_New, 
 IsUndersupplyBackordered_Old, 
 IsUndersupplyBackordered_New,
 Comments_Old, Comments_New, 
 DeliveryInstructions_Old, DeliveryInstructions_New, 
 InternalComments_Old, InternalComments_New, 
 PickingCompletedWhen_Old,
 PickingCompletedWhen_New, LastEditedBy_Old, 
 LastEditedBy_New, LastEditedWhen_Old, 
 LastEditedWhen_New, ActionType, ActionTime, UserName)
 SELECT
 ISNULL(Inserted.OrderID, Deleted.OrderID) AS OrderID,
 Deleted.CustomerID AS CustomerID_Old,
 Inserted.CustomerID AS CustomerID_New,
 Deleted.SalespersonPersonID AS SalespersonPersonID_Old,
 Inserted.SalespersonPersonID AS SalespersonPersonID_New,
 Deleted.PickedByPersonID AS PickedByPersonID_Old,
 Inserted.PickedByPersonID AS PickedByPersonID_New,
 Deleted.ContactPersonID AS ContactPersonID_Old,
 Inserted.ContactPersonID AS ContactPersonID_New,
 Deleted.BackorderOrderID AS BackorderOrderID_Old,
 Inserted.BackorderOrderID AS BackorderOrderID_New,
 Deleted.OrderDate AS OrderDate_Old,
 Inserted.OrderDate AS OrderDate_New,
 Deleted.ExpectedDeliveryDate 
 AS ExpectedDeliveryDate_Old,
 Inserted.ExpectedDeliveryDate 
 AS ExpectedDeliveryDate_New,
 Deleted.CustomerPurchaseOrderNumber 
 AS CustomerPurchaseOrderNumber_Old,
 Inserted.CustomerPurchaseOrderNumber 
 AS CustomerPurchaseOrderNumber_New,
 Deleted.IsUndersupplyBackordered 
 AS IsUndersupplyBackordered_Old,
 Inserted.IsUndersupplyBackordered 
 AS IsUndersupplyBackordered_New,
 Deleted.Comments AS Comments_Old,
 Inserted.Comments AS Comments_New,
 Deleted.DeliveryInstructions 
 AS DeliveryInstructions_Old,
 Inserted.DeliveryInstructions 
 AS DeliveryInstructions_New,
 Deleted.InternalComments AS InternalComments_Old,
 Inserted.InternalComments AS InternalComments_New,
 Deleted.PickingCompletedWhen 
 AS PickingCompletedWhen_Old,
 Inserted.PickingCompletedWhen 
 AS PickingCompletedWhen_New,
 Deleted.LastEditedBy AS LastEditedBy_Old,
 Inserted.LastEditedBy AS LastEditedBy_New,
 Deleted.LastEditedWhen AS LastEditedWhen_Old,
 Inserted.LastEditedWhen AS LastEditedWhen_New,
 CASE
 WHEN Inserted.OrderID IS NULL THEN 'DELETE'
 WHEN Deleted.OrderID IS NULL THEN 'INSERT'
 ELSE 'UPDATE'
 END AS ActionType,
 SYSUTCDATETIME() ActionTime,
 SUSER_SNAME() AS UserName
 FROM Inserted
 FULL JOIN Deleted
 ON Inserted.OrderID = Deleted.OrderID;
END
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

该触发器的唯一功能是将数据插入到日志表中，每行数据对应一个给定的写操作。它很简单，随着时间的推移易于记录和维护，表也会发生变化。如果需要跟踪其他详细信息，可以添加其他列，如数据库名称、服务器名称、受影响列的行数或调用的应用程序。

3. 最后一步是测试和验证日志表是否正确。

以下是添加触发器后对表进行更新的测试:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
UPDATE Orders
 SET InternalComments = 'Item is no longer backordered',
 BackorderOrderID = NULL,
 IsUndersupplyBackordered = 0,
 LastEditedBy = 1,
 LastEditedWhen = SYSUTCDATETIME()
FROM sales.Orders
WHERE Orders.OrderID = 10;
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

结果如下：

![](https://img-blog.csdnimg.cn/img_convert/e7badf4bfabc7ece67f39d1e4bc801b6.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

![](https://img-blog.csdnimg.cn/img_convert/4a8ada0a92d520b0c3b197906f9b0374.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

上面省略了一些列，但是我们可以快速确认已经触发了更改，包括日志表末尾新增的列。

**INSERT 和 DELETE**

前面的示例中，进行插入和删除操作后，读取日志表中使用的数据。这种特殊的表可以作为任何相关写操作的一部分。INSERT 将包含被插入操作触发，DELETE 将被删除操作触发，UPDATE 包含被插入和删除操作触发。

对于 INSERT 和 UPDATE，将包含表中每个列新值的快照。对于 DELETE 和 UPDATE 操作，将包含写操作之前表中每个列旧值的快照。

**触发器什么时候最有用**

DML 触发器的最佳使用是简短、简单且易于维护的写操作，这些操作在很大程度上独立于应用程序业务逻辑。

*   触发器的一些重要用途包括:
*   记录对历史表的更改
*   审计用户及其对敏感表的操作。
*   向表中添加应用程序可能无法使用的额外值 (由于安全限制或其他限制)，例如:
    *    登录 / 用户名
    *    操作发生时间
    *   服务器 / 数据库名称
*   简单的验证。

关键是让触发器代码保持足够的紧凑，从而便于维护。当触发器增长到成千上万行时，它们就成了开发人员不敢去打扰的黑盒。结果，更多的代码被添加进来，但是旧的代码很少被检查。即使有了文档，这也很难维护。

为了让触发器有效地发挥作用，应该将它们编写为基于设置的。如果存储过程必须在触发器中使用，则确保它们在需要时使用表值参数，以便可以基于集的方式移动数据。下面是一个触发器的示例，该触发器遍历 id，以便使用结果顺序 id 执行示例存储过程:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TRIGGER TR_Sales_Orders_Process
 ON Sales.Orders
 AFTER INSERT
AS
BEGIN
 SET NOCOUNT ON;
 DECLARE @count INT;
 SELECT @count = COUNT(*) FROM inserted;
 DECLARE @min_id INT;
 SELECT @min_id = MIN(OrderID) FROM inserted;
 DECLARE @current_id INT = @min_id;
 WHILE @current_id < @current_id + @count
 BEGIN
 EXEC dbo.process_order_fulfillment 
 @OrderID = @current_id;
 SELECT @current_id = @current_id + 1;
 END
END
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

虽然相对简单，但当一次插入多行时对 Sales.Orders 的 INSERT 操作的性能将受到影响，因为 SQL Server 在执行 process_order_fulfillment 存储过程时将被迫逐个执行。一个简单的修复方法是重写存储过程，并将一组 Order id 传递到存储过程中，而不是一次一个地这样做:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TYPE dbo.udt_OrderID_List AS TABLE(
 OrderID INT NOT NULL,
 PRIMARY KEY CLUSTERED 
( OrderID ASC));
GO
CREATE TRIGGER TR_Sales_Orders_Process
 ON Sales.Orders
 AFTER INSERT
AS
BEGIN
 SET NOCOUNT ON;
 DECLARE @OrderID_List dbo.udt_OrderID_List;
 EXEC dbo.process_order_fulfillment @OrderIDs = @OrderID_List;
END
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

更改的结果是将完整的 id 集合从触发器传递到存储过程并进行处理。只要存储过程以基于集合的方式管理这些数据，就可以避免重复执行，也就是说，避免在触发器内使用存储过程有很大的价值，因为它们添加了额外的封装层，进一步隐藏了在数据写入表时执行的 TSQL。它们应该被认为是最后的手段，只有当可以在应用程序的许多地方多次重写 TSQL 时才使用。

**什么时候触发器是危险的**

架构师和开发人员面临的最大挑战之一是确保触发器只在需要时使用，而不允许它们成为一刀切的解决方案。向触发器添加 TSQL 通常被认为比向应用程序添加代码更快、更容易，但随着时间的推移，这样做的成本会随着每添加一行代码而增加。

触发器在以下情况下会变得危险:

*   保持尽可能少的触发以减少复杂性。
*   触发代码变得复杂。如果更新表中的一行导致要执行数千行添加的触发器代码，那么开发人员就很难完全理解数据写入表时会发生什么。更糟糕的是，当出现问题时，故障排除非常具有挑战性。
*   触发器跨服务器。这将网络操作引入到触发器中，可能导致在出现连接问题时写入速度变慢或失败。如果目标数据库是要维护的对象，那么即使是跨数据库触发器也会有问题。
*   触发器调用触发器。触发器中最令人痛苦的是，当插入一行时，写操作会导致 75 个表中有 100 个触发器要执行。在编写触发器代码时，确保触发器可以执行所有必要的逻辑，而不会触发更多触发器。额外的触发通常是不必要的。
*   递归触发器被设置为 ON。这是一个默认设置为 off 的数据库级别设置。打开时，它允许触发器的内容调用相同的触发器。递归触发器会极大地损害性能，调试时也会非常混乱。通常，当一个触发器中的 DML 作为操作的一部分触发其他触发器时，使用递归触发器。
*   函数、存储过程或视图都在触发器中。在触发器中封装更多的业务逻辑会使它们变得更复杂，并给人一种触发器代码短小简单的错误印象，而实际上并非如此。尽可能避免在触发器中使用存储过程和函数。
*   迭代发生。循环和游标本质上是逐行操作的，可能会导致对 1000 行的操作一次触发 1000 次，这极大地损害了查询性能。

这是一个很长的列表，但通常可以总结为短而简单的触发器会表现得更好，并避免上面的大多数陷阱。如果使用触发器来维护复杂的业务逻辑，那么随着时间的推移，越来越多的业务逻辑将被添加进来，并且不可避免地将违反上述最佳实践。

重要的是要注意，为了维护原子的、事务，受触发器影响的任何对象都将保持事务处于打开状态，直到该触发器完成。这意味着长触发器不仅会使事务持续时间更长，而且还会持有锁并导致持续时间更长。因此，在测试触发器时，在为现有触发器创建或添加额外逻辑时，应该了解它们对锁、阻塞和等待的影响。

**如何改善触发器**

有很多方法可以使触发器更易于维护、更容易理解和性能更高。以下是一些关于如何有效管理触发器和避免落入陷阱的建议。

触发器本身应该有良好的文档记录:

*   这个触发器为什么存在?
*   它能做什么?
*   它是如何工作的?
*   对于触发器的工作方式是否有任何例外或警告?

此外，如果触发器中的 TSQL 难以理解，那么可以添加内联注释，以帮助第一次查看它的开发人员。

下面是触发器文档的样例:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
/* 12/29/2020 EHP
 This trigger logs all changes to the table to the Orders_log
 table that occur for non-internal customers.
 CustomerID = -1 signifies an internal/test customer and 
 these are not audited.
*/
CREATE TRIGGER TR_Sales_Orders_Audit
 ON Sales.Orders
 FOR INSERT, UPDATE, DELETE
AS
BEGIN
 SET NOCOUNT ON;
 INSERT INTO Sales.Orders_log
 (OrderID, CustomerID_Old, CustomerID_New, 
 SalespersonPersonID_Old, SalespersonPersonID_New,
 PickedByPersonID_Old, PickedByPersonID_New,
 ContactPersonID_Old, ContactPersonID_New, 
 BackorderOrderID_Old, BackorderOrderID_New, 
 OrderDate_Old, OrderDate_New, 
 ExpectedDeliveryDate_Old,
 ExpectedDeliveryDate_New, 
 CustomerPurchaseOrderNumber_Old, 
 CustomerPurchaseOrderNumber_New, 
 IsUndersupplyBackordered_Old, 
 IsUndersupplyBackordered_New,
 Comments_Old, Comments_New, 
 DeliveryInstructions_Old, DeliveryInstructions_New, 
 nternalComments_Old, InternalComments_New, 
 PickingCompletedWhen_Old, PickingCompletedWhen_New, 
 LastEditedBy_Old, LastEditedBy_New, 
 LastEditedWhen_Old, LastEditedWhen_New, 
 ActionType, ActionTime, UserName)
 SELECT
 ISNULL(Inserted.OrderID, Deleted.OrderID) AS OrderID, 
 -- The OrderID can never change. 
 --This ensures we get the ID correctly, 
 --regardless of operation type.
 Deleted.CustomerID AS CustomerID_Old,
 Inserted.CustomerID AS CustomerID_New,
 Deleted.SalespersonPersonID AS SalespersonPersonID_Old,
 Inserted.SalespersonPersonID AS SalespersonPersonID_New,
 Deleted.PickedByPersonID AS PickedByPersonID_Old,
 Inserted.PickedByPersonID AS PickedByPersonID_New,
 Deleted.ContactPersonID AS ContactPersonID_Old,
 Inserted.ContactPersonID AS ContactPersonID_New,
 Deleted.BackorderOrderID AS BackorderOrderID_Old,
 Inserted.BackorderOrderID AS BackorderOrderID_New,
 Deleted.OrderDate AS OrderDate_Old,
 Inserted.OrderDate AS OrderDate_New,
 Deleted.ExpectedDeliveryDate AS ExpectedDeliveryDate_Old,
 Inserted.ExpectedDeliveryDate AS ExpectedDeliveryDate_New,
 Deleted.CustomerPurchaseOrderNumber 
 AS CustomerPurchaseOrderNumber_Old,
 Inserted.CustomerPurchaseOrderNumber 
 AS CustomerPurchaseOrderNumber_New,
 Deleted.IsUndersupplyBackordered 
 AS IsUndersupplyBackordered_Old,
 Inserted.IsUndersupplyBackordered 
 AS IsUndersupplyBackordered_New,
 Deleted.Comments AS Comments_Old,
 Inserted.Comments AS Comments_New,
 Deleted.DeliveryInstructions 
 AS DeliveryInstructions_Old,
 Inserted.DeliveryInstructions 
 AS DeliveryInstructions_New,
 Deleted.InternalComments AS InternalComments_Old,
 Inserted.InternalComments AS InternalComments_New,
 Deleted.PickingCompletedWhen AS PickingCompletedWhen_Old,
 Inserted.PickingCompletedWhen 
 AS PickingCompletedWhen_New,
 Deleted.LastEditedBy AS LastEditedBy_Old,
 Inserted.LastEditedBy AS LastEditedBy_New,
 Deleted.LastEditedWhen AS LastEditedWhen_Old,
 Inserted.LastEditedWhen AS LastEditedWhen_New,
 CASE -- Determine the operation type based on whether 
 --Inserted exists, Deleted exists, or both exist.
 WHEN Inserted.OrderID IS NULL THEN 'DELETE'
 WHEN Deleted.OrderID IS NULL THEN 'INSERT'
 ELSE 'UPDATE'
 END AS ActionType,
 SYSUTCDATETIME() ActionTime,
 SUSER_SNAME() AS UserName
 FROM Inserted
 FULL JOIN Deleted
 ON Inserted.OrderID = Deleted.OrderID
 WHERE Inserted.CustomerID <> -1 
 -- -1 indicates an internal/non-production 
 --customer that should not be audited.
 OR Deleted.CustomerID <> -1; 
 -- -1 indicates an internal/non-production 
 --customer that should not be audited.
END
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

请注意，该文档并不全面，但包含了一个简短的头，并解释了触发器内的一些 TSQL 关键部分:

*   排除 CustomerID = -1 的情况。这一点对于不知道的人来说是不明显的，所以这是一个很好的注释。
*   ActionType 的 CASE 语句用于什么。
*   为什么在插入和删除之间的 OrderID 列上使用 ISNULL。

**使用 IF UPDATE**

在触发器中，UPDATE 提供了判断是否将数据写入给定列的能力。这可以允许触发器检查列在执行操作之前是否发生了更改。下面是该语法的示例:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TRIGGER TR_Sales_Orders_Log_BackorderID_Change
 ON Sales.Orders
 AFTER UPDATE
AS
BEGIN
 SET NOCOUNT ON;
 IF UPDATE(BackorderOrderID)
 BEGIN
 UPDATE OrderBackorderLog
 SET BackorderOrderID = Inserted.BackorderOrderID,
 PreviousBackorderOrderID = Deleted.BackorderOrderID
 FROM dbo.OrderBackorderLog
 INNER JOIN Inserted
 ON Inserted.OrderID = OrderBackorderLog.OrderID
 END
END
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

通过首先检查 BackorderID 是否被更新，触发器可以在不需要时绕过后续操作。这是一种提高性能的好方法，它允许触发器根据所需列的更新值完全跳过代码。

COLUMNS_UPDATED 指示表中的哪些列作为写操作的一部分进行了更新，可以在触发器中使用它来快速确定指定的列是否受到插入或更新操作的影响。虽然有文档记录，但它使用起来很复杂，很难进行文档记录。我通常不建议使用它，因为它几乎肯定会使不熟悉它的开发人员感到困惑。

请注意，对于 UPDATE 或 COLUMNS_UPDATED，列是否更改并不重要。对列进行写操作，即使值没有改变，对于 UPDATE 操作仍然返回 1，对于 COLUMNS_UPDATED 操作仍然返回 1。它们只跟踪指定的列是否是写操作的目标，而不跟踪值本身是否改变。

**每个操作一个触发器**

让触发代码尽可能的简单。数据库表的触发器数量增长会大大增加表的复杂性，理解其操作变得更加困难。。

例如，考虑以下表触发器定义方式:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TRIGGER TR_Sales_Orders_I
 ON Sales.Orders
 AFTER INSERT
CREATE TRIGGER TR_Sales_Orders_IU
 ON Sales.Orders
 AFTER INSERT, UPDATE
CREATE TRIGGER TR_Sales_Orders_UD
 ON Sales.Orders
 AFTER UPDATE, DELETE
CREATE TRIGGER TR_Sales_Orders_UID
 ON Sales.Orders
 AFTER UPDATE, INSERT, DELETE
CREATE TRIGGER TR_Sales_Orders_ID
 ON Sales.Orders
 AFTER INSERT, DELETE
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

当插入一行时会发生什么? 触发器的触发顺序是什么? 这些问题的答案需要研究。维护更少的触发器是一个简单的解决方案，并且消除了对给定表中如何发生写操作的猜测。作为参考，可以使用系统存储过程 sp_settriggerorder 修改触发器顺序，不过这只适用于 AFTER 触发器。

**再简单一点**

触发器的最佳实践是操作简单，执行迅速，并且不会因为它们的执行而触发更多的触发器。触发器的复杂程度并没有明确的规则，但有一条简单的指导原则是，理想的触发器应该足够简单，如果必须将触发器中包含的逻辑移到其他地方，那么迁移的代价不会高得令人望而却步。也就是说，如果触发器中的业务逻辑非常复杂，以至于移动它的成本太高而无法考虑，那么这些触发器很可能变得过于复杂。

使用我们前面的示例，考虑一下更改审计的触发器。这可以很容易地从触发器转移到存储过程或代码中，而这样做的工作量并不大。触发器中记录日志的方便性使它值得一做，但与此同时，我们应该知道开发人员将 TSQL 从触发器迁移到另一个位置需要多少小时。

时间的计算可以看作是触发器的可维护性成本的一部分。也就是说，如果有必要，为摆脱触发机制而必须付出的代价。这听起来可能很抽象，但平台之间的数据库迁移是很常见的。在 SQL Server 中执行良好的一组触发器在 Oracle 或 PostgreSQL 中可能并不有效。

**优化表变量**

有时，一个触发器中需要临时表，以允许对数据进行多次更新。临时表存储在 tempdb 中，并且受到 tempdb 数据库大小、速度和性能约束的影响。

对于经常访问的临时表，优化表变量是在内存中 (而不是在 tempdb 中) 维护临时数据的好方法。

下面的 TSQL 为内存优化数据配置了一个数据库 (如果需要):

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
ALTER DATABASE WideWorldImporters 
SET MEMORY_OPTIMIZED_ELEVATE_TO_SNAPSHOT = ON;
ALTER DATABASE WideWorldImporters ADD FILEGROUP WWI_InMemory_Data 
 CONTAINS MEMORY_OPTIMIZED_DATA;
ALTER DATABASE WideWorldImporters ADD FILE 
 (NAME='WideWorldImporters_IMOLTP_File_1', 
 FILENAME='C:\SQLData\WideWorldImporters_IMOLTP_File_1.mem') 
 TO FILEGROUP WWI_InMemory_Data;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

一旦配置完成，就可以创建一个内存优化的表类型:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TYPE dbo.SalesOrderMetadata
AS TABLE
( OrderID INT NOT NULL PRIMARY KEY NONCLUSTERED,
 CustomerID INT NOT NULL,
 SalespersonPersonID INT NOT NULL,
 ContactPersonID INT NOT NULL,
 INDEX IX_SalesOrderMetadata_CustomerID NONCLUSTERED HASH 
 (CustomerID) WITH (BUCKET_COUNT = 1000))
WITH (MEMORY_OPTIMIZED = ON);
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

这个 TSQL 创建了演示的触发器所需要的表:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TABLE dbo.OrderAdjustmentLog
( OrderAdjustmentLog_ID int NOT NULL IDENTITY(1,1) 
 CONSTRAINT PK_OrderAdjustmentLog PRIMARY KEY CLUSTERED,
 OrderID INT NOT NULL,
 CustomerID INT NOT NULL,
 SalespersonPersonID INT NOT NULL,
 ContactPersonID INT NOT NULL,
CreateTimeUTC DATETIME2(3) NOT NULL);
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

下面是一个使用内存优化表的触发器演示:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE TRIGGER TR_Sales_Orders_Mem_Test
 ON Sales.Orders
 AFTER UPDATE
AS
BEGIN
 SET NOCOUNT ON;
 DECLARE @OrderData dbo.SalesOrderMetadata;
 INSERT INTO @OrderData
 (OrderID, CustomerID, SalespersonPersonID, 
 ContactPersonID)
 SELECT
 OrderID,
 CustomerID,
 SalespersonPersonID,
 ContactPersonID
 FROM Inserted;
 
 DELETE OrderData
 FROM @OrderData OrderData
 INNER JOIN sales.Customers
 ON Customers.CustomerID = OrderData.CustomerID
 WHERE Customers.IsOnCreditHold = 0;
 UPDATE OrderData
 SET ContactPersonID = 1
 FROM @OrderData OrderData
 WHERE OrderData.ContactPersonID IS NULL;
 
 INSERT INTO dbo.OrderAdjustmentLog
 (OrderID, CustomerID, SalespersonPersonID, 
 ContactPersonID, CreateTimeUTC)
 SELECT
 OrderData.OrderID,
 OrderData.CustomerID,
 OrderData.SalespersonPersonID,
 OrderData.ContactPersonID,
 SYSUTCDATETIME()
 FROM @OrderData OrderData;
END
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

触发器内需要的操作越多，节省的时间就越多，因为内存优化的表变量不需要 IO 来读 / 写。

一旦读取了来自所插入表的初始数据，触发器的其余部分就可以不处理 tempdb，从而减少使用标准表变量或临时表的开销。

下面的代码设置了一些测试数据，并运行一个更新来演示上述代码的结果:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
UPDATE Customers
 SET IsOnCreditHold = 1
FROM Sales.Customers
WHERE Customers.CustomerID = 832;
UPDATE Orders
 SET SalespersonPersonID = 2
FROM sales.Orders
WHERE CustomerID = 832;
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

一旦执行，OrderAdjustmentLog 表的内容可以被验证:

![](https://img-blog.csdnimg.cn/img_convert/c58d11e2bd6fe11e97f7d5acc0b98b04.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​

结果是意料之中的。通过减少对标准存储的依赖并将中间表移动到内存中，内存优化表提供了一种大大提高触发速度的方法。这仅限于对临时对象有大量调用的场景，但在存储过程或其他过程性 TSQL 中也很有用。

**替代触发器**

像所有的工具一样，触发器也可能被滥用，并成为混乱、性能瓶颈和可维护性噩梦的根源。有许多比触发器更可取的替代方案，在实现 (或添加到现有的) 触发器之前应该考虑它们。

**Temporal tables**

Temporal tables 是在 SQL Server 2016 中引入的，它提供了一种向表添加版本控制的简单方法，无需构建自己的数据结构和 ETL。这种记录对应用程序是不可见的，并提供了符合 ANSI 标准的完整版本支持，使之成为一种简单的方法来解决保存旧版本数据的问题。

**Check 约束**

对于简单的数据验证，Check 约束可以提供所需的内容，而不需要函数、存储过程或触发器。在列上定义 Check 约束，并在创建数据时自动验证数据。

下面是一个 Check 约束的示例:

```
ALTER TABLE Sales.Invoices WITH CHECK ADD CONSTRAINT
CK_Sales_Invoices_ReturnedDeliveryData_Must_Be_Valid_JSON
CHECK ([ReturnedDeliveryData] IS NULL OR 
ISJSON([ReturnedDeliveryData])<>(0))
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这段代码检查一个列是否是有效的 JSON。如果是，则执行正常进行。如果不是，那么 SQL Server 将抛出一个错误，写操作将失败。Check 约束可以检查列和值的任何组合，因此可以管理简单或复杂的验证任务。

创建 Check 约束的成本不高，而且易于维护。它们也更容易记录和理解，因为 Check 约束的范围仅限于验证传入数据和确保数据完整性，而触发器实际上可以做任何可以想象的事情!

**唯一约束**

如果一个列需要唯一的值，并且不是表上的主键，那么唯一约束是完成该任务的一种简单而有效的方法。唯一约束是索引和唯一性的组合。为了有效地验证唯一性，索引是必需的。

下面是一个唯一约束的例子:

```
ALTER TABLE Warehouse.Colors ADD CONSTRAINT 
UQ_Warehouse_Colors_ColorName UNIQUE NONCLUSTERED (ColorName ASC);
```

每当一行被插入到 Warehouse.Colors 表中，将检查 ColorName 的唯一性。如果写操作碰巧导致了重复的颜色，那么语句将失败，数据将不会被更改。为此目的构建了唯一约束，这是在列上强制唯一性的最简单方法。

内置的解决方案将更高效、更容易维护和更容易记录。任何看到唯一约束的开发人员都将立即理解它的作用，而不需要深入挖掘 TSQL 来弄清事情是如何工作的，这种简单性使其成为理想的解决方案。

**外键约束**

与 Check 约束和唯一约束一样，外键约束是在写入数据之前验证数据完整性的另一种方式。外键将一一表中的列链接到另一张表。当数据插入到目标表时，它的值将根据引用的表进行检查。如果该值存在，则写操作正常进行。如果不是，则抛出错误，语句失败。

这是一个简单的外键例子:

```
ALTER TABLE Sales.Orders WITH CHECK ADD CONSTRAINT
FK_Sales_Orders_CustomerID_Sales_Customers FOREIGN KEY (CustomerID)
REFERENCES Sales.Customers (CustomerID);
```

当数据写入 Sales.Orders 时，CustomerID 列将根据 Sales.Customers 中的 CustomerID 列进行检查。

与唯一约束类似，外键只有一个目的: 验证写入一个表的数据是否存在于另一个表中。它易于文档化，易于理解，实现效率高。

触发器不是执行这些验证检查的正确位置，与使用外键相比，它是效率较低的解决方案。

**存储过程**

在触发器中实现的逻辑通常可以很容易地移动到存储过程中。这消除了大量触发代码可能导致的复杂性，同时允许开发人员更好的维护。存储过程可以自由地构造操作，以确保尽可能多的原子性。

实现触发器的基本原则之一是确保一组操作与写操作一致。所有成功或失败都是作为原子事务的一部分。应用程序并不总是需要这种级别的原子性。如果有必要，可以在存储过程中使用适当的隔离级别或表锁定来保证事务的完整性。

虽然 SQL Server(和大多数 RDBMS) 提供了 ACID 保证事务将是原子的、一致的、隔离的和持久的，但我们自己代码中的事务可能需要也可能不需要遵循相同的规则。现实世界的应用程序对数据完整性的需求各不相同。

存储过程允许自定义代码，以实现应用程序所需的数据完整性，确保性能和计算资源不会浪费在不需要的数据完整性上。

例如，一个允许用户发布照片的社交媒体应用程序不太可能需要它的事务完全原子化和一致。如果我的照片出现在你之前或之后一秒，没人会在意。同样，如果你在我编辑照片的时候评论我的照片，时间对使用这些数据的人来说可能并不重要。另一方面，一个管理货币交易的银行应用程序需要确保交易是谨慎执行的，这样就不会出现资金丢失或数字报告错误的情况。如果我有一个银行账户，里面有 20 美元，我取出 20 美元的同时，其他人也取出了 20 美元，我们不可能都成功。我们中的一个先得到 20 美元，另一个遇到关于 0 美元余额的适当错误消息。

**函数**

函数提供了一种简单的方法，可以将重要的逻辑封装到一个单独的位置。在 50 个表插入中重用的单个函数比 50 个触发器 (每个表一个触发器) 执行相同逻辑要容易得多。

考虑以下函数:

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
CREATE FUNCTION Website.CalculateCustomerPrice
 (@CustomerID INT, @StockItemID INT, @PricingDate DATE)
RETURNS DECIMAL(18,2)
WITH EXECUTE AS OWNER
AS
BEGIN
 DECLARE @CalculatedPrice decimal(18,2);
 DECLARE @UnitPrice decimal(18,2);
 DECLARE @LowestUnitPrice decimal(18,2);
 DECLARE @HighestDiscountAmount decimal(18,2);
 DECLARE @HighestDiscountPercentage decimal(18,3);
 DECLARE @BuyingGroupID int;
 DECLARE @CustomerCategoryID int;
 DECLARE @DiscountedUnitPrice decimal(18,2);
 SELECT @BuyingGroupID = BuyingGroupID,
 @CustomerCategoryID = CustomerCategoryID
 FROM Sales.Customers
 WHERE CustomerID = @CustomerID;
 SELECT @UnitPrice = si.UnitPrice
 FROM Warehouse.StockItems AS si
 WHERE si.StockItemID = @StockItemID;
 SET @CalculatedPrice = @UnitPrice;
 SET @LowestUnitPrice = (
 SELECT MIN(sd.UnitPrice)
 FROM Sales.SpecialDeals AS sd
 WHERE ((sd.StockItemID = @StockItemID) 
 OR (sd.StockItemID IS NULL))
 AND ((sd.CustomerID = @CustomerID) 
 OR (sd.CustomerID IS NULL))
 AND ((sd.BuyingGroupID = @BuyingGroupID) 
 OR (sd.BuyingGroupID IS NULL))
 AND ((sd.CustomerCategoryID = @CustomerCategoryID) 
 OR (sd.CustomerCategoryID IS NULL))
 AND ((sd.StockGroupID IS NULL) OR EXISTS (SELECT 1 
 FROM Warehouse.StockItemStockGroups AS sisg
 WHERE sisg.StockItemID = @StockItemID
 AND sisg.StockGroupID = sd.StockGroupID))
 AND sd.UnitPrice IS NOT NULL
 AND @PricingDate BETWEEN sd.StartDate AND sd.EndDate);
 IF @LowestUnitPrice IS NOT NULL AND @LowestUnitPrice < @UnitPrice
 BEGIN
 SET @CalculatedPrice = @LowestUnitPrice;
 END;
 SET @HighestDiscountAmount = (
 SELECT MAX(sd.DiscountAmount)
 FROM Sales.SpecialDeals AS sd
 WHERE ((sd.StockItemID = @StockItemID) 
 OR (sd.StockItemID IS NULL))
 AND ((sd.CustomerID = @CustomerID) 
 OR (sd.CustomerID IS NULL))
 AND ((sd.BuyingGroupID = @BuyingGroupID) 
 OR (sd.BuyingGroupID IS NULL))
 AND ((sd.CustomerCategoryID = @CustomerCategoryID) 
 OR (sd.CustomerCategoryID IS NULL))
 AND ((sd.StockGroupID IS NULL) OR EXISTS 
 (SELECT 1 FROM Warehouse.StockItemStockGroups AS sisg 
 WHERE sisg.StockItemID = @StockItemID
 AND sisg.StockGroupID = sd.StockGroupID))
 AND sd.DiscountAmount IS NOT NULL
 AND @PricingDate BETWEEN sd.StartDate AND sd.EndDate);
 IF @HighestDiscountAmount IS NOT NULL AND (
 @UnitPrice - @HighestDiscountAmount) < @CalculatedPrice
 BEGIN
 SET @CalculatedPrice = @UnitPrice - @HighestDiscountAmount;
 END;
 SET @HighestDiscountPercentage = (
 SELECT MAX(sd.DiscountPercentage)
 FROM Sales.SpecialDeals AS sd
 WHERE ((sd.StockItemID = @StockItemID)
 OR (sd.StockItemID IS NULL))
 AND ((sd.CustomerID = @CustomerID) 
 OR (sd.CustomerID IS NULL))
 AND ((sd.BuyingGroupID = @BuyingGroupID) 
 OR (sd.BuyingGroupID IS NULL))
 AND ((sd.CustomerCategoryID = @CustomerCategoryID) 
 OR (sd.CustomerCategoryID IS NULL))
 AND ((sd.StockGroupID IS NULL) OR EXISTS 
 (SELECT 1 FROM Warehouse.StockItemStockGroups AS sisg
 WHERE sisg.StockItemID = @StockItemID
 AND sisg.StockGroupID = sd.StockGroupID))
 AND sd.DiscountPercentage IS NOT NULL
 AND @PricingDate BETWEEN sd.StartDate AND sd.EndDate);
 IF @HighestDiscountPercentage IS NOT NULL
 BEGIN
 SET @DiscountedUnitPrice = ROUND(@UnitPrice * 
 @HighestDiscountPercentage / 100.0, 2);
 IF @DiscountedUnitPrice < @CalculatedPrice 
 SET @CalculatedPrice = @DiscountedUnitPrice;
 END;
 RETURN @CalculatedPrice;
END;
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

就复杂性而言，这绝对是一头猛兽。虽然它接受标量参数来确定计算价格，但它执行的操作非常大，甚至包括对 Warehouse.StockItemStockGroups, Warehouse.StockItems 和 Sales.Customers 的额外读取。如果这是一个经常针对单行数据使用的关键计算，那么将其封装在一个函数中是获得所需计算的一种简单方法，而不会增加触发器的复杂性。小心使用函数，并确保使用大型数据集进行测试。简单的标量函数通常可以很好地伸缩性较大的数据，但更复杂的函数可能性能较差。

**编码**

当从应用程序修改表中的数据时，还可以在写入数据之前执行额外的数据操作或验证。这通常代价低廉，性能很好，并有助于减少失控触发器对数据库的负面影响。

将代码放入触发器的常见理由是，这样做可以避免修改代码、推送构建，否则会导致更改应用程序。这与在数据库中进行更改相关的任何风险直接相反。这通常是应用程序开发人员和数据库开发人员之间关于谁将负责新代码的讨论。

这是一个粗略的指导方针，但有助于在代码添加到应用程序或触发器之后测量可维护性和风险。

**计算列**

其他列发生更改时，计算列可以包括通过各种各样的算术运算和函数进行计算，得到结果。它们可以包含在索引中，也可以包含在唯一的约束中，甚至主键中。

当任何底层值发生变化时，SQL Server 会自动维护计算的列。注意，每个计算出来的列最终都是由表中其他列的值决定的。

这是使用触发器来维护指定列值的一种很好的替代方法。计算列是高效的、自动的，并且不需要维护。它们只是简单地工作，甚至允许将复杂的计算直接集成到一个表中，而在应用程序或 SQL Server 中不需要额外的代码。

**使用 SQL Server 触发器**

触发器在 SQL Server 中是一个有用的特性，但像所有工具一样，它也可能被误用或滥用。在决定是否使用触发器时，一定要考虑触发器的目的。

如果一个触发器被用来将简短的事务数据写入日志表，那么它很可能是一个很好的触发器。如果触发器被用来强制执行复杂的业务规则，那么很可能需要重新考虑处理这类操作的最佳方式。

有很多工具可以作为触发器的可行替代品，比如检查约束、计算列等，解决问题的方法并不短缺。数据库体系结构的成功在于为工作选择正确的工具。

原文链接：[https://www.red-gate.com/simple-talk/sql/database-administration/sql-server-triggers-good-scary/](https://www.red-gate.com/simple-talk/sql/database-administration/sql-server-triggers-good-scary/)

![](https://img2020.cnblogs.com/blog/2248030/202102/2248030-20210225085610626-1554359937.jpg)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​