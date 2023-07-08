# Ali_Khatami_Solidity15
### Advanced Solidity Custom errors(learning from the video of Pattrick collins)
we wil create custom error here to save gas
```

//SPDX-License-Identifier:MIT

pragma solidity ^0.8.8;

import "./akrkPriceConvertor.sol";


error NotOwner();


contract akrkFundMe  {

    using akrkPriceConverter for uint256;
    //if you assing a variable outside of a function and never change it then use constant keyword like below

    uint256 public  constant  MINIMUM_USD=50 * 1e18; //as getConversionRate() returns 18 zeroes after decimel place
 
    address[] public funders;// we create an address array make it public 
// we have use payable keyword with the function below to make it payable with any native blockchain currency
    //below we have done mapping of addresses to how much money each of the people sent
    mapping(address => uint256) public addressToAmountFunded ;


    address public immutable  owner;// this is a global variable



    //the constructor below will set up as the owner of the contract

     constructor(){

         owner= msg.sender; // owner is equal to msg.sender

     }


    function fund() public payable {
     
     //as we want to convert msg.value to USD we made the following changes
       
        require(msg.value.getConversionRate() >= MINIMUM_USD , "Not send enough");// to get accees to the 'value' at deploy at run transaction tab
        //1e18 is 1 ether, here the value must be greater than 1 ether
        //require is a checker it checks whether the value is greater than 1 ether
        // if not it is going to revert with an error messsage shown above
        //adding funders to the array 
         funders.push(msg.sender);
         addressToAmountFunded[msg.sender] = msg.value;// It is for when somebody funds our contract

    }// To send money. We made it public so that anybody can call it

  //The function below is created so that we can get price of ethereum in terms of USD , so that we can convert msg.value to USD
//Both functions are public because we can do whatever we want with them
//By using the getPrice() function below we are going to interact with contract outside of our project
  

  //below we will create a withdraw function

  //we use modifier in the function below

  function withdraw() public onlyOwner{

      

      for(uint256 funderIndex=0; funderIndex < funders.length; funderIndex++){

          address funder = funders[funderIndex]; //to access the first element or 0th element and gonna return an address for us to use
          // we gonna use the above line to reset our mapping 
          addressToAmountFunded[funder]=0;
      }

      // reset the array
      //here below the funders array is now equal to brand new address array with zero object to start with
      // if 1 in the first bracket then there will be one element to start with
      funders= new address[](0);
      // if we want to transfer fund to whomever is calling the withdrawal function we wil do the 
      //following
     // msg.sender.transfer(address(this).balance); //here 'this' refers to the whole 'akrkFundMe' contract
      // by using .balance we can get ethereum blockchain currency of this contract address 

      

      //call is the recommended as it forwards all gas and it has no limit and bool like sender

      (bool callSuccess, )= payable(msg.sender).call{value: address(this).balance}("");//here inside the bracket of call we gonna call function but now it is kept blank
     // here it gonna returned two variables but we need only one varaible
     require(callSuccess, "Call failed");


     
}


modifier onlyOwner {
         //require(msg.sender == owner, "Sender is not owner!");// to check is msg.sender is equal to owner

         if(msg.sender != owner) { revert NotOwner(); }
         _;
    
     }

  
   
    

}

```


### Receive and fall back

What happens if someone send or contract eth without calling the fund function<br>
For that we need some function to keep track of the people hwo send eth. there are two special functions for that<br>
one is called receive() and another is aclled fallback() <br>

![a6](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/ce950adc-055c-4125-9b5b-76f10b5367e2)

figure1; we will create a new solidiy file which is ```akrkFallbackExample.sol```  and write vthe code below:

```
//SPDX-License-Identifier:MIT

pragma solidity ^0.8.8;

contract akarkfallbackExample{

    uint256 public result;

    receive() external payable{

        result=1;


    }
}

```

thewe will get the following output <br>

![a7](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/7a997a58-30af-403a-a48a-d1b1c2ae9b57)
Figure2: here after clicking the ```result``` button it shows zero because we have not initialized anything <br>

![a8](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/36628526-bca6-4cc4-9da5-e6b677cfbf32)
Figure3: we can send this contract , ethereum directly by working with this ```low level interaction```<br>


![a9](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/966b7791-e25f-49c4-8a84-0511d3d684cd)
Figure4; in this arear of low level interaction we acn work with differnt function<br>

if we keep the ```calldata``` blank it will be the same as if we are clicking ```Send``` on ```metamask``` wallet<br>

![a10](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/76b862d2-dd84-4334-a3b9-dffabde64710)
figure5: if we put ```1``` in the value field and click the ```Transact``` button we see successful transaction occured 

![a11](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/e38d22d9-18ce-4db2-920f-45af30089781)
Figure6: here at console we can see that it has called our receive function<br>

![a12](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/3b96e5b2-9c87-4641-b4ba-7e963f76c979)
figure7: here we can see after we click the ```result``` button we see it has now become ```1```<br>

![a13](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/83765a2c-a375-4ecc-8703-16a97a898aaf)
Figure8: if we keep the value equal to zero and click the ```transact``` button and then click ```result``` button<br>
it will also give ouput as ```1```<br>

our receive function gets executed anytime we send a transaction to this now and we don't specify a function and we keep the calldata blank<br>

When working with any other contract like ```akrkFundme.sol``` when we call one of the functions at the contract we are actually just inserting <br>
this call data bit with certain data that points to one of those functions up there <br>

![a14](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/61105c62-1488-408d-9bf7-fb77baec7d84)
Figure9: now if we send any data at calldata field solidity says ```Fallback function is not defined``` this is<br>
because whenever we send any data at calldata field, ```Remix``` is smart enough to know that we not looking for receive<br>
we are looking for ```fallback``` and when it does not find any ```fallback``` function in the code it shows this error <br>

```fallback``` function sis similiar to ```receive``` function except for the fact that can work even when data is sent along with transaction

In ```fallback``` and ```receive``` function we will not use the function keyword.<br>











