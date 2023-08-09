SISMO CONNECT 


Introduction :

Sismo uses zero-knowledge proofs (ZKPs) to let users prove ownership/group membership of their Data Sources. Personal data (e.g. data that proves ownership of an NFT from a specific collection) can be revealed to applications integrated with Sismo Connect without revealing the associated Data Source



Sismo Connect Configuration

The SismoConnectConfiguration is the first object to understand when starting Sismo Connect integration in your application. Its main feature is to reference the appId you got from the Sismo Factory when you created a new Sismo Connect app. It will then enable you to request proofs from your users and verify them.
Type definitions

The SismoConnectConfiguration is an object with the following properties:
appId (required): the Sismo Connect application identifier that you have created on the Sismo Factory.
vault.impersonate (optional): by default set to null. The list of Data Sources that you want to impersonate. They will be automatically added to your developer Vault.
displayRawResponse (optional): by default set to false. If set to true, the Sismo Data Vault app will display the proof generated in a modal and will not redirect to your application once the proof is generated. It can be useful in development.
sismoApiUrl (optional): API endpoint to fetch group information.
vaultAppBaseUrl (optional): by default set to https://vault-beta.sismo.io/.


Auths

Sismo Connect can be used to authenticate a user from multiple sources, either from web2 or web3.

Type definitions

The AuthRequest is an object with the following properties:
AuthType (required): defines the type of authentication required. The following authType are currently supported:
VAULT: Sismo Connect returns the user's vaultId for the application that requested it. It is a deterministic anonymous identifier (hash(userVaultSecret, AppId, ..)) See more information about Vault Identifiers here.
GITHUB: Sismo Connect returns the user's GitHub account id.
TWITTER Sismo Connect returns the user's Twitter account id.
EVM_ACCOUNT: Sismo Connect returns the user's Ethereum address.
TELEGRAM : Sismo Connect returns the user's Telegram account id.
userId (optional): requests the user to have a predefined account.
isOptional(optional): by default set to false. Allows the user to optionally authenticate with this AuthType.

Claims

Type definitions

The ClaimRequest holds all the information needed to generate such a proof. It has the following properties:
groupId (required): the group identifier that the user must prove membership of in order to generate the proof. The groupId can be found in the Sismo Factory.
value (optional): by default set to “1”. Querying a specific value restricts eligibility to users belonging to the group with the minimum specified value or matching the exact value.
claimType (optional): by default ClaimType.GTE. Defines the type of group membership required. The following claim types are currently supported:
ClaimType.GTE: the user must prove that they own an account with at least the requested value.
ClaimType.EQ: the user must prove that he owns an account with the exact requested value.
groupTimestamp (optional): set to “latest” by default. Groups are composed of snapshots generated either once, daily, or weekly. Each Group Snapshot generated has a timestamp associated with them. By default, the selected group is the latest Group Snapshot generated. But you are free to select a Group Snapshot with a different timestamp than the latest generated one.
isOptional(optional): by default set to false. If set to true the user can optionally prove their group membership.
isSelectableByUser(optional): by default set to false and only available for ClaimType.GTE. This allows users to selectively choose the value used to generate the proof


Signature

A Signature is a specific message embedded in a generated proof that will be checked when verifying the proof. It can be requested from the Data Vault via a SignatureRequest.

Why a signature is often needed when verifying proofs onchain?

The SignatureRequest is an object with the following properties:
The signed message is not mandatory when you interact with your contracts, but it is very often needed. As far as your users are generating valid proofs, it could be quite easy for a third party to front-run them by just taking their proof and making their own call to your smart contracts with it.
To overcome this issue, we offer a way to embed a specific message in a proof. This way it can be thought of as a signature since this proof could not be valid without checking successfully that the signed message is correct onchain.
For example, it is mandatory to request the user to embed the address where they want to receive an airdrop in a proof. If a third party takes the proof, the call will be reverted with a signature mismatch message, effectively protecting your users from being front-run.
Example: You are a DAO voting platform, and you want to get the vote of a user onchain. To do so, the user will create a signature containing a message (his vote) that will be used to generate the proof. When verifying the proof onchain, you are also verifying that the proof contains the message. It is then impossible for a malicious actor to take your proof and vote for something else.

Type definitions

message (required): the message that the user is going to sign.
isSelectableByUser(optional): by default set to false. Allows the user to edit the message in the Sismo Data Vault app.




Vault Identifiers
App-specific anonymous user ID.

Sismo uses zero-knowledge proofs (ZKPs) to let users prove ownership/group membership of their Data Sources. Personal data (e.g. data that proves ownership of an NFT from a specific collection) can be revealed to applications integrated with Sismo Connect without revealing the associated Data Source.
For example, users can prove they own a certain NFT (i.e. prove they are part of the group of NFT owners) without revealing the wallet address (i.e. Data Source) holding it.
Despite offering privacy, many applications still need to keep track of individual users to avoid double spends or enable more complex user management. Identifiers also need to remain specific to each application to avoid the possibility of tracking users across multiple applications.
Sovereign & Anonymous ID for Sismo Connect Apps
A Vault Identifier, also known as vaultId, is calculated using the following formula:
vaultId = hash(vaultSecret, hash(appId, derivationKey))
Here's what each component represents:
1.vaultSecret: This is a confidential piece of information exclusive to a Data Vault’s owner. it must be understood as the seed/ private key of the user.
2.appId: This represents the unique identifier for the associated Sismo Connect application.
3.derivationKey: By default, this is set to zero. However, it can optionally be used by application developers to generate multiple user identifiers for a single Data Vault owner.


Anonymous
Only the owner of a Data Vault knows their associated vaultSecret. As a result, they are the only person capable of computing their unique vaultId for a specific application.
Computation occurs when users prove group memberships/ownership of Data Sources during the generation of ZKPs. When the application verifies the ZKP, it receives the Vault Identifier as an output, thus authenticating the owner as a unique user.

App-Specific
The use of an application's appId in the formula ensures the uniqueness of Vault Identifiers across different applications. This feature eliminates any risk of cross-application exposure or doxxing.

Deterministic
As a Vault Identifier is deterministically derived from a vaultSecret and appId , a single user cannot have multiple identifiers for a specific application unless they create another Data Vault. However, Data Sources can only be added to a single Data Vault, thus ensuring a user cannot use the same Data Source twice to prove membership in a Data Group without the application being aware.



Native Data Sources
In addition, Vault Identifiers can be used as native Data Sources in the Sismo ecosystem. This allows for the creation of Data Groups containing Vault Identifiers, which can be used by users to make group membership claims on additional Sismo Connect applications.

Vault Identifier as a Nullifier

SafeAirdrop, a Sybil-resistant airdrop leveraging privately aggregated data, uses vaultId to prevent individual users from receiving an airdrop more than once. As users always compute the same vaultId on a specific application, they cannot access the application’s gated feature after already claiming the airdrop.
Although the application can identify whether a user is unique or not, privacy is not compromised as a vaultId does not contain any sensitive information and cannot be linked to other applications or the user’s private vault. In this context, vaultId acts as the nullifier frequently used by zero-knowledge protocols to prevent double spends.

Vault Identifier as a Sybil-Resistant User ID
Privacy Is Normal is an anonymous Sybil-resistant lottery for Tornado Cash users. The application uses Vault Identifiers to identify lottery participants are unique individuals and select winners. Lottery winners were then able to claim their prize by proving their own a winning vaultId.
Despite knowing whether a user has already entered the lottery or not, the application has no access to sensitive information that could identify the user in question. As a result, vaultId functions as a Sybil-resistant user ID that allows the application to track user activity without infringing on their privacy.





