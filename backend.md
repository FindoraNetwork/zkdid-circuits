
On-chain contracts:

1) The issuer will deploy an NFT (SBT) contract "QuickKYC ZKCred V1 (QZK)"

   Deploy a non-transferable NFT smart contract (zkCredential issuer will be the owner) to EVM with following APIs
    - Use Openzeppelin `ERC721Mintable` (https://docs.openzeppelin.com/contracts/2.x/api/token/erc721#ERC721Mintable)
      as base and remove transfer-related methods.

    - `constructor` will take single parameter of zkCredential type (e.g., "https://ld.findora.org/creddential-kyc/v1").
      constructor(string credType)

    - `mint` function will take 3 parameters
      mint(address to, uint256 tokenId, string commitment)

    - `credType` returns zk credential type
      credType() public view returns (string _credType)

    - `commitment` returns zkCredential's commitment
      commitment(uint256 tokenId) public view returns (string commitment)


2) The issuer also deploys smart contracts to provide zk proof verification service.
   The two contract:
     - https://github.com/FindoraNetwork/zkdid-circuits/blob/main/src/circuits/build/zkcredit/zkcredit_700_js/zkcredit_650_verifier.sol
     - https://github.com/FindoraNetwork/zkdid-circuits/blob/main/src/circuits/build/zkcredit/zkcredit_700_js/zkcredit_700_verifier.sol
     - `verifyProof` returns true only when proof is valid AND matching the `input`
       verifyProof(
            uint[2] memory a,
            uint[2][2] memory b,
            uint[2] memory c,
            uint[2] memory input
        ) public view returns (bool r)


3) The verifier deploys a smart contract "ZKVerifier" to EVM with following APIs
    - `whiteList` takes 3 parameters for whitelisting:
      `credType` (e.g., "https://ld.findora.org/creddential-kyc/v1")
      `token` (the SBT token address)
      `verifier` (the one deployed by issuer)
      whiteList(string credType, address token, address verifier)

      Note: A "ZKVerifier" can whitelist many verifiers to support different types of zk credentials (and different checks) issued by different credential issuers.

    - `credTypes` returns full list of whitelisted credential types.
      credTypes() public view returns (string[] memory)

    - `verify` checks token, proof passed by the user according to credType
      verify(string credType, uint256 tokenId, proof) {
         // check and see if credType is whitelisted or not
         // check ownership of the tokenId if whitelisted
         // get commitment by tokenId
         // call whitelisted verifier's `verifyProof` function with the commitment (the public.json https://github.com/FindoraNetwork/zkdid-circuits/blob/main/src/circuits/build/zkcredit/zkcredit_650_js/public.json#L2)
         // return true/false
    }


Off-chain server:

1) Host an credential issuer server (SpruceID HTTP Server)
   https://www.spruceid.dev/didkit/didkit-packages/http-server

  1.1) Host an SIEW (if have time) to verify ownership of Metamask wallet
     https://github.com/spruceid/siwe-quickstart

2) A zkCredential issue server to issue SBT to user's Metamask address
   // address: user metamask
   // cred: the credential struct
   // return: transaction Hash
   fn (address: string, cred: Credential) -> string




Demo credential:
transparent: https://github.com/FindoraNetwork/zkdid-circuits/blob/main/creds/credential-signed.jsonld#L22
zk: https://github.com/FindoraNetwork/zkdid-circuits/blob/main/creds/zk-credential-signed.jsonld#L3


Dome credential encoding:

{
  dob:    12546000  ==> HEX:   00000000000000000000000000BF6FD0
                    ==> Input: 12546000

  name:   Bob       ==> HEX:   00000000000000000000000000426F62
                    ==> Input: 4353890

  ssn:    433543937 ==> HEX:   00000000000000000000000019D75B01
                    ==> Input: 433543937

  credit: 702       ==> HEX:   000000000000000000000000000002BE
                    ==> Input: 702
}

DATA: 00000000000000000000000000BF6FD000000000000000000000000000426F6200000000000000000000000019D75B01000000000000000000000000000002BE
HASH: 8d9d6f7e9202abc8d7143a7562bb9e8299c75c454c9d79669328cca5615e0ae0
