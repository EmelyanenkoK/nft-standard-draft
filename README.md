# NFT Standard
## State
 **Draft** to not be used in production. Example collection can be seen [https://emelyanenkok.github.io/nft-standard-draft/](here).

## Acknowledgments
Standard is greatly based on TonWhales [https://github.com/tonwhales/ton-nft](ton-nft) code. Thanks @Naltox, @ex3ndr for fruitful discussions.

## Architecture
### NFT
Each NFT is separate smart-contract which has 
- owner: actor which can transfer ownership and also control TON on contract
- editor: (maybe absent) actor which can edit nft content and code
- collection: (maybe absent) contract which contain common part of the content and can be used for indexing all nft in collection
- content: standart does not restrict form of content, although there are some suggestions

Owner controls NFT via internal messages. Upon receiving `transfer` request from current owner, NFT updates ownership information in storage. Additionally `transfer` request may contain `payload` and TONs which will be transferred to new owner for notification or more complex actions (for instance trigger auction start)

Editor controls NFT via internal messages as well. Editor has `transfer_editorship` (with capability to send notification to new editor as well), `update_code` and `update_content` commands.

There is rule of content represention that if collection is defined, total content, uri and other parameters are calculated on collection contract from individual content of the NFT and common content of whole collection

Minimal balance of NFT is 0.05 TON, this limit may be increased for NFT storing huge content data, but should not be lowered.
#### Get-methods
- `get_nft_data` returns `init`, `index`, `collection_address`, `owner_address`, `editor_address`, `individual_content`
   * `init`- 0 or -1 indicate whether this contract is initialised
   * `index` - represent sequential index of NFT in collection
   * `collection_address` - pair of integers containing workchain and address of the collection
   * `owner_address` - pair of integers containing workchain and address of the owner
   * `editor_address` - pair of integers containing workchain and address of the editor
   * `individual_content` - cell with content
- `get_name` (optional) returns `slice name`, containing string with name of the token.
- `get_symbol` (optional) returns `slice symbol`, containing string with symbol of the token.
- `uri` (optional) returns `slice uri`, containing string with uri of the token.

### Collection
Collection contract is special wallet-type contract with ability to deploy NFTs. Addresses of NFT are calculatbale by collection and only collection can initialise NFT with content, ownership and other information.
Minimal balance of Collection is 1 TON, this limit may be increased for NFT storing huge content data, but should not be lowered.

#### Get-methods
- `get_nft_address_by_index` and `get_std_nft_address_by_index` return address of the token with given index.
- `get_next_index` return next index to be deployed (equal to number of exemplars in collection)
- `member_content` return cell with content from given index and individual content of the nft
- `member_uri` return slice with content from given index and individual content of the nft
- `royalty_params` return `numerator`, `denominator`, and `address` for royalty (not forcible onchain)


Note all string in getmethod responses are organized as follows: it should be slices with at most one reference; if reference is presented it content concantenated to the end of parent slice; this procedure is recursively applyed to reference as well.
