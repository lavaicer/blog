![](https://raw.githubusercontent.com/lavaicer/Img/main/202206061736118.png)

# 研究论文：对 Web 用户帐户的预劫持攻击

[MSRC / Andrew Paverd / 2022 年 5 月 23 日/攻击、纵深防御、缓解措施、安全研究](https://msrc-blog.microsoft.com/2022/05/23/pre-hijacking-attacks/)

> 译者注:
> 经典认证:使用个人邮箱注册
> 联合认证:第三方授权注册 - 例如使用 GIthub 注册

2020 年，MSRC 授予了两项[独立项目研究资助](https://www.microsoft.com/en-us/msrc/grant-microsoft-identity)，以支持外部研究人员致力于进一步加强身份协议和系统的安全性。今天，我们很高兴地发布这些项目中的第一个项目的结果。这项由独立安全研究员 Avinash Sudhodanan 领导的研究调查了帐户预劫持——一种影响网站和其他在线服务的新型攻击。

与经典的帐户劫持攻击类似，攻击者的目标是获得对受害者帐户的访问权限。但是，如果攻击者可以在受害者创建帐户之前使用受害者的电子邮件地址在目标服务上创建一个帐户，则攻击者可以使用各种技术将帐户置于预劫持状态。在受害者恢复访问权限并开始使用该帐户后，攻击者可以重新获得访问权限并接管该帐户。

这项工作的全部细节可在我们的研究论文中找到，该论文将在 8 月的第 31 届 USENIX 安全研讨会上发表。在本文中，我们描述了五种类型的帐户预劫持攻击，并且我们表明大量网站和在线服务可能容易受到这些攻击。因此，我们发表这项研究的主要动机是提高对这些攻击的认识并帮助组织防御它们。

## 用户帐户创建中的挑战

用户帐户是网站和其他在线服务的普遍特征，因此已成为攻击者的宝贵目标。帐户劫持是一种众所周知的威胁，攻击者试图在未经授权的情况下访问受害者的用户帐户。

组织正确地投入大量资源来防御帐户劫持。但是，较少受到关注的一个方面是用户帐户的创建过程，以及相应的安全隐患。随着向联合身份和单点登录 (SSO) 的发展，许多服务现在支持（至少）两种不同的用户创建帐户的路径：提供用户名和密码的经典路径和使用身份提供者的联合路径（ IdP）例如[使用 Microsoft 登录](https://developer.microsoft.com/en-us/identity/add-sign-in-with-microsoft)。创建帐户后，某些服务还提供链接 IdP 帐户的可能性，以便用户可以直接登录或通过 IdP 进行身份验证。

以前的学术研究已经讨论过“抢先式帐户劫持攻击”，攻击者可以控制受害者的 IdP 帐户，并使用它在受害者尚未注册的服务上创建用户帐户。受此攻击的启发，我们证明存在一整类此类攻击，我们将其称为帐户预劫持攻击。与之前的工作相比，我们的攻击都不需要攻击者破坏受害者的 IdP 帐户。

## 帐户预劫持攻击

帐户预劫持攻击的显着特征是攻击者在受害者在目标服务上创建帐户之前执行一些操作。毫无戒心的受害者可能随后重新获得对该帐户的访问权并开始使用它，可能会添加个人信息、付款详细信息或任何其他类型的私人信息。一段时间后，攻击者通过获得对受害者帐户的访问权限来完成攻击——基本上实现了与帐户劫持攻击相同的目标。

在研究论文中，我们描述了五种类型的预劫持攻击：

1. Classic-Federated Merge Attack：利用经典和联合认证之间交互的潜在弱点来创建帐户。攻击者使用受害者的电子邮件地址通过经典认证创建一个帐户，然后受害者通过联合认证创建一个帐户，使用相同的电子邮件地址。如果服务不安全地合并这两个帐户，则可能导致受害者和攻击者都可以访问同一帐户。
2. Unexpired Session Identifier Attack：这利用了一个漏洞，在该漏洞中，当用户重置密码时，经过身份验证的用户不会退出帐户。攻击者使用受害者的电子邮件地址创建一个帐户，然后保持一个长时间运行的活动会话。当受害者恢复帐户时，如果密码重置没有使攻击者的会话无效，攻击者可能仍然具有访问权限。
3. 木马标识符攻击：这利用了攻击者将附加身份标识符链接到使用经典用户名和密码创建的帐户的可能。攻击者使用受害者的电子邮件地址创建一个帐户，然后将木马标识符（例如攻击者的联合身份或另一个攻击者控制的电子邮件地址或电话号码）添加到该帐户。当受害者重置密码时，攻击者可以使用木马标识符来访问该帐户（例如通过重置密码或请求一次性登录链接）。
4. 未过期的电子邮件更改攻击：这利用了一个潜在的漏洞，即当用户重置密码期间,服务无法使电子邮件更改功能确定的验证 URL 无效。攻击者使用受害者的电子邮件地址创建一个帐户，并开始将帐户的电子邮件地址更改为攻击者自己的电子邮件地址。作为此过程的一部分，该服务通常会向攻击者的电子邮件发送一个验证 URL，但攻击者尚未确认更改。在受害者恢复帐户并开始使用后，攻击者完成更改电子邮件过程以控制该帐户。
5. Non-Verifying IdP Attack：这种攻击是 Classic-Federated Merge Attack 的镜像。攻击者利用在创建联合认证身份时不验证电子邮件地址所有权的 IdP。使用此非验证 IdP，攻击者使用目标服务创建一个帐户，并等待受害者使用经典方式创建一个帐户。如果服务根据电子邮件地址错误地组合了这两个帐户，攻击者将能够访问受害者的帐户。
   ![img](https://raw.githubusercontent.com/lavaicer/Img/main/202206061733252.png)
   对于所有这些攻击，攻击者需要识别受害者尚未拥有帐户但可能在未来创建帐户的服务。尽管不能保证成功，但攻击者可以通过多种方式解决此问题。例如，在个人层面上，攻击者可能已经知道特定受害者使用哪些服务，并机会主义地预先劫持其他类似或相关服务的帐户。更广泛地说，攻击者可能会了解到整个组织（例如，大学系）计划使用特定服务，并且可以预先劫持来自该组织的任何公开电子邮件地址的帐户。或者，攻击者可能会观察到服务受欢迎程度的普遍增加（例如，当人们需要在家工作时的视频会议服务）并使用通过网站抓取或凭证转储找到的电子邮件地址预先劫持该服务的帐户。如果受害者已经在服务中创建了一个帐户，那么攻击者通常不会有任何风险。
   ![img](https://raw.githubusercontent.com/lavaicer/Img/main/202206061734649.png)

## 实证评估

为了确定易受攻击服务的普遍性，我们分析了 75 个最受欢迎的网站和在线服务，发现其中至少 35 个容易受到一种或多种预劫持攻击，包括广泛使用的云存储、社交和专业网络、博客和视频会议服务。遵循协同漏洞披露 (CVD)原则，我们在 2021 年 3 月至 2021 年 9 月期间向受影响的组织报告了这些漏洞。但是，除了我们分析的 75 个网站和在线服务之外，其他网站和在线服务很可能也容易受到这些攻击。因此，我们发布这项研究以阐明此类漏洞，以便任何运营网站或其他服务的组织都可以采取措施保护其用户帐户。
在我们工作的同时，其他研究人员也展示了类似的攻击（有时称为“账户前接管”攻击）。一些示例包括：对不安全的电子邮件确认 URL 参数进行逆向工程 [例如 craighayes ] 或查找不安全地合并通过经典路由和联合认证创建的帐户的服务，如果它们使用相同的（可能未经验证的）电子邮件地址 [例如 hackerone，hbothra22 ]。这些例子说明了这些漏洞可能有多广泛。

## 根本原因和缓解措施

从根本上说，帐户预劫持漏洞的根本原因是服务未能在允许使用帐户之前验证用户是否确实拥有所提供的标识符（例如电子邮件地址或电话号码）。尽管许多服务需要标识符验证，但它们通常是异步进行的，允许用户（或攻击者）在验证标识符之前使用帐户的某些功能。虽然这可能会提高可用性，但它会为预劫持攻击创造一个漏洞窗口。

如果服务向用户提供的电子邮件地址发送验证电子邮件并要求在允许对帐户进行任何进一步操作之前完成验证，则上述所有攻击都可以得到缓解。类似的方法可用于验证其他类型标识符的所有权，例如使用短信或自动语音呼叫来确认电话号码的所有权。如果服务使用 IdP，它应该检查 IdP 是执行此验证还是执行自己的附加验证。

## 纵深防御

认识到在所有服务中实施严格的标识符验证可能需要时间，我们还确定了一套用于帐户创建的纵深防御安全措施，这也将减轻上述攻击。

密码重置：重置帐户密码时，服务必须执行以下操作：

1. 注销所有其他会话并使该帐户的所有其他身份验证令牌无效，以减轻 Unexpired Session 攻击。
2. 取消该帐户的所有待处理电子邮件更改操作，以减轻未过期电子邮件更改攻击。
3. 通知用户哪些联合身份、电子邮件地址和电话号码与帐户相关联，并要求用户选择他们不识别的任何标识符（即默认保留），或者更优选地选择哪些标识符保留（即默认取消链接）。

合并帐户：当服务将通过经典路由创建的帐户与通过联合路由创建的帐户合并（反之亦然）时，服务必须确保用户当前控制这两个帐户。例如，当用户尝试通过联合路由创建帐户但同一电子邮件地址已存在经典帐户时，应要求用户提供或重置经典帐户的密码。这将减轻 Classic-Federated Merge 和 Non-verifying IdP 攻击。

电子邮件更改确认：当服务发送确认电子邮件地址更改的能力（例如，带有嵌入式身份验证令牌的代码或 URL）时，此能力的有效期应尽可能低，在可用性的限制内, 以最小化 Unexpired Email Change 攻击的漏洞窗口。但是，这不会阻止攻击者不断请求新功能，因此服务应该限制可以为未经验证的标识符请求新功能的次数。

未经验证的帐户修剪：定期删除未经验证的帐户将减少大多数预劫持攻击的漏洞窗口（非验证 IdP 攻击除外）。但是，这不会阻止攻击者创建具有相同标识符的新帐户。因此，该服务应监控并可能限制为同一未经验证的标识符创建新帐户的次数。但是，这可用于通过耗尽合法用户标识符的帐户创建配额来发起拒绝服务 (DoS) 攻击。因此，该服务可以改为降低未经验证帐户的修剪阈值，并利用机器人检测框架来限制攻击者自动创建新帐户的速率。

多因素身份验证 (MFA)：用户可以通过尽快在其帐户上激活 MFA 来保护自己免受预劫持攻击。正确实施的 MFA 将阻止攻击者在受害者开始使用此帐户后对预劫持帐户进行身份验证。该服务还必须使在激活 MFA 之前创建的任何会话无效，以防止 Unexpired Session 攻击。

## 结论

总之，我们希望这项研究将有助于阐明这一潜在广泛存在的漏洞类别，并帮助组织实施有效的缓解策略。
– Avinash Sudhodanan（独立研究员）和 Andrew Paverd（MSRC）
