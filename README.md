# Ali_Khatami_Solidity15
### Advanced Solidity Custom errors(learning from the video of Pattrick collins)
we wil create custom error here to save gas

```solidity

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

In ```fallback``` and ```receive``` function we will not use the function keyword as it is expected by solidity.<br>

```constructor``` function is another special function where we don't use the function keyword<br>

solidity knows that this ```constructor``` function is immediately called when we deploy the contract<br>


![b1](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/02387998-fb86-4b2a-a561-03f2bc33f453)
Figure10: when we declare fallback function in the code and hit the transact button by typing the data <br>
no such error is shown raher transaction occured. here it is sending data to our contract <br>
It is equivalent to calling our contract withot any valid function <br>

![b2](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/11d61709-71ea-40e9-878d-55cdb8678e11)

Figure11: Now if we don't type anything in the transact field solidity will understand we gonna send <br>
ethereum or call this contract without specifying what we want to do and if we click the transact button <br>
transaction will occur and then click the result button it update back to 1 <br>

![b3](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/066de972-23d1-40f7-b8fe-b1f760254758)
Figure12: Here in the photo we can see that solidity by example dot org have wonderfuul chart that we can use<br>
to figure out whether or not ```receive``` function is going to be called or ```fallback``` function is going<br>
to be called . Source: https://youtu.be/gyMwXuJrbJQ



![b4](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/e0543fc4-2100-4a1c-a318-80d3f13d0e16)

We can use these ```receive``` and ```fallback``` functions in our akrkFundme contract when someone actually one wants <br>
to send us money instead of calling the fund function correctly 


we will add the ```receive``` function if somebody accidently send it money we can still proceeds the transaction<br>


The code is given below and for the libraray code for the code is here at https://github.com/C191068/Ali_Khatami_Solidity_11.git

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

  receive() external payable {
      fund();

  }

  fallback() external payable {
      fund();
      
  }   
    

}

```

if anyone sends any transaction without calling the fund function then in the abvove code ```receive() external payable``` <br>
will automatically route them to the fund function <br>

If anyone does not send us enough funding it will still get reverted <br>

![b5](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/89dff193-a9d1-4497-aa04-288461149a5c)
Figure14: here we change the environment to ```injected-provider metamask``` i.e we have connected our metamask wallet account<br>

![b6](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/06f2064c-dec2-450f-a8b0-51392d00e615)
Figure15: when we click the ```Deploy``` button the metamask window pop up asking for confirmation of transaction<br>

![b7](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/5a96768e-6c0a-42c6-ace6-85b137b6997e)
Figure16: at last succesful transaction have occured<br>

![b8](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/b25848ad-6e80-4358-9416-744c200e1609)
Figure17: this is our deployed contract 

![b9](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/842f782a-dcb1-4d41-9f73-1a37feba5e6e)
Figure18: here we can see when click the owner bbutton it gives the same address as our metamask wallet<br>

![b11](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/7350d482-ba40-4f10-b618-3f75496eea80)
Figure19: here when we click the MinimumUSD button it shows the minimum USD and in the fund button when <br>
we click 0 it is showing that nothing is funded so it is a blank contract<br>

![b12](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/749681b7-ac63-4df6-8736-81fe95b9d16c)
Figure20: when we copy the address and paste it on sepolia etherscan we see that the balance is 0 eth and <br>
transaction occured due to contract creation <br>

![b13](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/dade7ab8-acc4-4231-8a77-088b7aede5c6)
Figure21: We now want to send this contract money without calling the fund function for that at first<br>
we copy the contract address again<br>

like previous we have to do calculation before sending money to this contract<br>

![b19](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/a05c7005-44e8-4ec3-87ef-a00a3adc3c3b)
Figure22: going to this link https://data.chain.link/ethereum/mainnet/crypto-usd/eth-usd we see the current value<br>
of ethereum in USD <br>

here minimum USD is 50 and then = 50/1865.53 = 0.026+0.006 = 0.03 minimum Sepolia Eth 

![b20](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/c6e87903-8a64-46fd-84bf-3241743e6cef)
Figure23: then we will click the send button <br>

![b21](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/84227a61-8570-426d-aee1-5741970bf575)
Figure24: at send option we will paste the address of our contract and at amount section we will typed <br>
0.03 Sepolia Eth and click the ```next``` button

![b23](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/4bc00220-5434-4147-89e9-fe1cdeaedbf3)
Figure25: Thus we have successfully send money to the contract without calling the fund function<br>
![b24](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/9ad12ff4-61b1-460e-9d2b-732d19955fc1)
Figure26: on the etherscan we can see the amount is also updated and on the transaction side the method is 
```transfer``` instead of ```fund``` <br>

![b25](https://github.com/C191068/Ali_Khatami_Solidity15/assets/89090776/95faa11f-6beb-4805-972d-e007ba1674b6)
Figure27: Now going to our deployed contract we see that when we typed 0 at the funder button field <br>
it gives us the address of the account which is sending money i.e the metamask wallet address. Then when<br>
at the addresstoAmountFunded field we paste address of the account it shows the amount of SepoliaEth send<br>








































