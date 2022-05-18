// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/security/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MineRioLand is ERC721, Pausable, Ownable {

/**
* EVENTS
*/

//this event will be called when a purchase is made successfully  so the front-side shows the required modal
event BuyFinished();



/**
* BARIABLES
*/
    /**
     * the land info struct 
     */
    struct LandInfo {
        uint tokenId;
        uint cityCode;
        uint district;
        uint landType;
        uint blocksCount;
        uint number;
        bool canBuy;
    }


    /**
     * this boolean can activate/deactivate the approval requests
     * this can be used to disable selling the NFTs in the second hand markets
     */
    bool private _canApprove = false;
    
    /**
     * this boolean can activate/deactivate the transfer requests
     * this can be used ti disable transfering the NFTs 
     */
    bool private _canTransfer = false;
    
    /**
     * this contract is pre-sale.
     * the main contract should have `retire` permission for this contract
     * so after the main land contract is deployed, 
     * the addres should be set here
     */
    address private _mainLandContract;

    /**
     * the adderss which holds the NFTs minted by this contract
     */
    address private _nftOwnerWallet;

    /**
     * the adderss which holds the payment tokens after the nft is bought from a wallet
     */
    address private _tokenOwnerWallet;

    /**
     * the base price of every block inside the map (by wei)
     */
    uint private _basePrice;
    
    /**
     * the current bnb price (by usd)
     */
    uint private _bnbPrice;
    
    /**
     * the district factorr for each district. (*1,000,000)
     */
    mapping(uint256 => uint) private _districtFactorPB;

    // Mapping from token ID to approved lands
    mapping(uint256 => LandInfo) private _landInfo;


    constructor() ERC721("MineRio Land Pre-Sale", "RIO") {
        _nftOwnerWallet = owner();
        _tokenOwnerWallet = owner();
    }

    function _baseURI() internal pure override returns(string memory) {
        return "https://minerio.net/api/token/get-info/";
    }

    /**
     * @dev See {IERC721-setApprovalForAll}.
     * the check permission for `_canApprove` is added
     */
    function setApprovalForAll(address operator, bool approved) public virtual override {
        require(_canApprove, 'you cant approve NFTs from this contract');
        super.setApprovalForAll(operator, approved);
    }

    /**
     * @dev See {IERC721-setApprovalForAll}.
     * the check permission for `_canApprove` is added
     */
    function approve(address to, uint256 tokenId) public virtual override {
        require(_canApprove, 'you cant approve NFTs from this contract');
        super.approve(to, tokenId);
    }

    /**
     * the pause function which will pause everything
     */
    function pause() public onlyOwner {
        _pause();
    }

    /**
     * the unpause function which will unpause everything
     */
    function unpause() public onlyOwner {
        _unpause();
    }

    /**
     * the mint function which takes a land 
     * only called by owner
     */
    function safeMint(LandInfo memory _land) public onlyOwner {
        require(validateLand(_land));
        _safeMint(_nftOwnerWallet, _land.tokenId);
        _landInfo[_land.tokenId] = _land;
    }

    /**
     * the bulk mint function which takes array lands
     * only called by owner
     */
    function safeBulkMint(LandInfo[] memory _lands) public onlyOwner {
        for(uint i=0;i<_lands.length ;i++){
            require(validateLand(_lands[i]));
            _safeMint(_nftOwnerWallet, _lands[i].tokenId);
            _landInfo[_lands[i].tokenId] = _lands[i];
        }
    }

    function _mint(address to, uint256 tokenId) internal override {
        super._mint(to, tokenId);
    }

    /**
     * validates the land to make sure the sent land from the owner is in currect format
     */
    function validateLand(LandInfo memory _land) internal pure returns(bool){
        return _land.tokenId!=0 && _land.cityCode!=0 &&  _land.district!=0 &&  _land.landType!=0 &&  _land.blocksCount!=0;
    }

    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
    internal
    whenNotPaused
    override {
        super._beforeTokenTransfer(from, to, tokenId);
    }


    /**
     * @dev See {IERC721-transferFrom}.
     * added _canTransfer checking
     * added _isApprovedOrOwner checking
     */
    function transferFrom(
        address from,
        address to,
        uint256 tokenId
    ) public virtual override {
        //solhint-disable-next-line max-line-length
        require(_canTransfer || msg.sender == owner(), 'you cant transfer your nft from this contract');
        require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: transfer caller is not owner nor approved");

        _transfer(from, to, tokenId);
    }

    /**
     * @dev See {IERC721-safeTransferFrom}.
     * added _canTransfer checking
     */
    function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId
    ) public virtual override {
        require(_canTransfer || msg.sender == owner(), 'you cant transfer your nft from this contract');
        safeTransferFrom(from, to, tokenId, "");
    }
    
    /**
     * @dev See {IERC721-safeTransferFrom}.
     * added _canTransfer checking
     * added _isApprovedOrOwner checking
     */
    function safeTransferFrom(
        address from,
        address to,
        uint256 tokenId,
        bytes memory _data
    ) public virtual override {
        require(_canTransfer || msg.sender == owner(), 'you cant transfer your nft from this contract');
        require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: transfer caller is not owner nor approved");
        _safeTransfer(from, to, tokenId, _data);
    }

    /**
     * the buy function which is payable
     * user will call this and should send exact value with the lands price
     */
    function buy(uint256 _id) external payable {
        // e.g. the buyer wants 100 tokens, needs to send 500 wei
        require(_landInfo[_id].canBuy, 'the canBuy is false');
        require(msg.value == priceOf(_id), 'Need to send exact amount of currency');
        require(msg.sender != ERC721.ownerOf(_id), 'already the owner');
        // send all Ether to owner
        // Owner can receive Ether since the address of owner is payable
        payable(_tokenOwnerWallet).transfer(msg.value);

        _landInfo[_id].canBuy = false;
        /*
         * sends the requested amount of tokens
         * from this contract address
         * to the buyer
         */
        _transfer(ERC721.ownerOf(_id), msg.sender, _id);

        //emit the result of buy
        emit BuyFinished();
    }

    /**
     * the function that sends value from current contract to the _tokenOwnerWallet
     */
    function payToOwner(uint val) public onlyOwner {
        payable(_tokenOwnerWallet).transfer(val);
    }


    /**
     * calculates the price of the land and returns
     */
    function priceOf(uint256 _tokenId) public view virtual returns(uint256) {
        require(_exists(_tokenId), "ERC721Metadata: URI query for nonexistent token");

        return ((_basePrice/_bnbPrice) * _landInfo[_tokenId].blocksCount * districtFactorPB(_landInfo[_tokenId].district) / 1000000);
    }

    /**
     * returns the _basePrice
     */
    function basePrice() public view virtual returns(uint256) {
        return _basePrice;
    }

    /**
     * check if user can buy the specific land
     */
    function canBuyOf(uint256 _tokenId) external view virtual returns(bool) {
        require(_exists(_tokenId), "ERC721Metadata: URI query for nonexistent token");

        return _landInfo[_tokenId].canBuy;
    }

    /**
     * returns the `_canApprove`
     */
    function canApprove() external view virtual returns(bool) {
        return _canApprove;
    }

    /**
     * returns the `_canTransfer`
     */
    function canTransfer() external view virtual returns(bool) {
        return _canTransfer;
    }

    /**
     * returns the `_mainLandContract`
     */
    function mainLandContract() external view virtual returns(address) {
        return _mainLandContract;
    }

    /**
     * returns the `_nftOwnerWallet`
     */
    function nftOwnerWallet() external view virtual returns(address) {
        return _nftOwnerWallet;
    }

    /**
     * returns the `_tokenOwnerWallet`
     */
    function tokenOwnerWallet() external view virtual returns(address) {
        return _tokenOwnerWallet;
    }

    /**
     * returns the `_bnbPrice`
     */
    function bnbPrice() external view virtual returns(uint) {
        return _bnbPrice;
    }

    /**
     * gets the id and returns the land info
     */
    function getLand(uint _id)external view returns(LandInfo memory){
        return _landInfo[_id];
    }


    
    /**
     * gets the _districtFactorPB of a _districtId
     * if the district does not exists in the mapping, 
     * the function returns 1000000 by default (100%)
     */
    function districtFactorPB(uint _districtId) public view virtual returns(uint) {
        if(_districtFactorPB[_districtId]==0)
            return 1000000;
        return _districtFactorPB[_districtId];
    }

    /**
     * sets the setDistrictFactorPB by billion (default is 1,000,000 (100%))
     */
    function setDistrictFactorPB(uint _districtId,uint _priceFactorPB) external virtual onlyOwner {
        _districtFactorPB[_districtId] = _priceFactorPB;
    }



    /**
     * setting the bnb price (the price should be by USD)
     */
    function setBnbPrice( uint256 _price) external virtual onlyOwner {
        _bnbPrice = _price;
    }

    /**
     * setting the base price (the price should be by wei)
     */
    function setBasePrice( uint256 _price) external virtual onlyOwner {
        _basePrice = _price;
    }

    /**
     * setting the base price (the price should be by wei) and the bnb price (the price should be by USD)
     */
    function setBnbPriceAndBasePrice( uint256 _price_bnb, uint256 _price_base) external virtual onlyOwner {
        _basePrice = _price_base;
        _bnbPrice = _price_bnb;
    }


    /**
     * setting canbuy for an specific NFT 
     * returns error if NFT id does not exist
     */
    function setCanBuy(uint256 tokenId, bool canBuy) external virtual onlyOwner {
        require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");

        _landInfo[tokenId].canBuy = canBuy;
    }
    
    /**
     * setting _canApprove 
     */
    function setCanApprove(bool can_approve) external virtual onlyOwner {
        _canApprove = can_approve;
    }

    /**
     * setting _canTransfer value 
     */
    function setCanTransfer(bool can_transfer) external virtual onlyOwner {
        _canTransfer = can_transfer;
    }

    /**
     * setting the main land contract
     */
    function setMainLandContract(address addr) external virtual onlyOwner {
        _mainLandContract = addr;
    }
    /**
     * setting the Nft owner wallet
     */
    function setNftOwnerWallet(address addr) external virtual onlyOwner {
        _nftOwnerWallet = addr;
    }

    /**
     * setting the token selling income owner wallet
     */
    function setTokenOwnerWallet(address addr) external virtual onlyOwner {
        _tokenOwnerWallet = addr;
    }


    /**
     * @dev Returns whether `spender` is allowed to manage `tokenId`.
     *
     * Requirements:
     *
     * - `tokenId` must exist.
     */
    function _isApprovedOrOwner(address spender, uint256 tokenId) internal view virtual override returns (bool) {
        require(_exists(tokenId), "ERC721: operator query for nonexistent token");
        address owner = ERC721.ownerOf(tokenId);
        return (spender == owner || (getApproved(tokenId) == spender && _canApprove) || (isApprovedForAll(owner, spender) && _canApprove));
    }


    
    /**
     * @dev See {IERC721-isApprovedForAll}.
     * added _canApprove checking
     */
    function isApprovedForAll(address owner, address operator) public view virtual override returns (bool) {
        return super.isApprovedForAll(owner,operator) && _canApprove;
    }



    
    /**
     * the burn funvtion 
     * only can be called by owner or the main land contract 
     * CAUTION: this function will burn the nft completely
     */
    function retire(uint256 tokenId) external virtual {
        require(msg.sender == owner() || msg.sender == _mainLandContract, 'the sender must by either owner or mainLandContract');
        delete _landInfo[tokenId];
        _burn(tokenId);
    }
}
