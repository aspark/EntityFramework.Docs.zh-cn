---
title: "客户端 vs。Server 评估版的 EF 核心"
author: rowanmiller
ms.author: divega
ms.date: 10/27/2016
ms.assetid: 8b6697cc-7067-4dc2-8007-85d80503d123
ms.technology: entity-framework-core
uid: core/querying/client-eval
ms.openlocfilehash: e1852b780041e9e92fb4d25129175346e3a601a3
ms.sourcegitcommit: 01a75cd483c1943ddd6f82af971f07abde20912e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2017
---
# <a name="client-vs-server-evaluation"></a>客户端 vs。Server 评估版

实体框架核心支持正在评估客户端和推送到数据库的某些部分上的查询部分。 负责要确定查询的哪些部分将计算在数据库中的数据库提供程序。

> [!TIP]  
> 你可以查看这篇文章[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)GitHub 上。

## <a name="client-evaluation"></a>客户端评估

下面的示例在一个帮助器方法用于对 Url 进行标准化适用于 SQL Server 数据库中返回的博客。 因为 SQL Server 提供程序没有深入了解如何实现此方法，则不可以将其转换为 SQL。 所有其他方面的查询将计算在数据库中，但传递返回`URL`通过这种方法在客户端上执行。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs?highlight=6)] -->
``` csharp
var blogs = context.Blogs
    .OrderByDescending(blog => blog.Rating)
    .Select(blog => new
    {
        Id = blog.BlogId,
        Url = StandardizeUrl(blog.Url)
    })
    .ToList();
```

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs)] -->
``` csharp
public static string StandardizeUrl(string url)
{
    url = url.ToLower();

    if (!url.StartsWith("http://"))
    {
        url = string.Concat("http://", url);
    }

    return url;
}
```

## <a name="disabling-client-evaluation"></a>禁用客户端评估

虽然客户端评估可非常有用，在某些情况下可能导致性能不佳。 请考虑以下查询中，现在筛选器中使用的帮助器方法的位置。 因为这不能在数据库中执行，则将所有数据拉入内存，然后选择筛选器应用于客户端。 根据的数据，以及多少该数据会被筛选掉量，这可能导致性能不佳。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs)] -->
``` csharp
var blogs = context.Blogs
    .Where(blog => StandardizeUrl(blog.Url).Contains("dotnet"))
    .ToList();
```

默认情况下，EF 核心执行客户端评估时将记录一个警告。 请参阅[日志记录](../miscellaneous/logging.md)有关查看日志记录输出的详细信息。 客户端评估发生引发或不执行任何操作时，你可以更改的行为。 这是通常在设置您的上下文中的选项时`DbContext.OnConfiguring`，或在`Startup.cs`如果正在使用 ASP.NET Core。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/ThrowOnClientEval/BloggingContext.cs?highlight=5)] -->
``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=EFQuerying;Trusted_Connection=True;")
        .ConfigureWarnings(warnings => warnings.Throw(RelationalEventId.QueryClientEvaluationWarning));
}
```
