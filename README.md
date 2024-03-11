# Modified-RCH-
This is RCH code compiled and executed on g++
#include "Ring.h"
#include <cstdio>
#include "crc32c_sse42_u64.h"
#include<iostream>

void Ring::insert(uint32_t nodeID){
    for(uint32_t i = 0; i < V; i++){
        uint32_t key2 = gen32bitRandNumber(i);
        uint32_t hashValue = crc32c_sse42_u64(nodeID, key2);
     //   printf(" Hash of Node is %d, \n", hashValue);
        workSet[hashValue] = nodeID;
      //  std::cout<<" Node "<< nodeID <<" hashed at location " <<hashValue <<std::endl;
        //printf(" Node: %d, hashed at location: %d  \n", nodeID, hashValue);
        // printf("+1\n");
    }
}

Ring::Ring(uint32_t w, uint32_t v): V(v), failedSet(), workSet(){
    uint32_t i = 0;
    for(i = 0; i < w; i++){
        insert(i);
    }
}

void Ring::updateRemoval(uint32_t nodeID){
    for(uint32_t i = 0; i < V; i++){
        uint32_t key2 = gen32bitRandNumber(i);
        uint32_t hashValue = crc32c_sse42_u64(nodeID, key2);
        workSet.erase(hashValue);
    }
    failedSet.push(nodeID);
}

uint32_t Ring::updateAddition(){
    uint32_t nodeID = failedSet.top();
    failedSet.pop();
    insert(nodeID);
    return 1;  //original code there was no return
}

uint32_t Ring::getNodeID(uint32_t K, uint32_t* numHash){
    uint32_t key2 = gen32bitRandNumber(K);
    uint32_t Khash = crc32c_sse42_u64(K, key2);
  //  printf(" Hash of Key is %u, \n", Khash);
    auto pNode = workSet.lower_bound(Khash);
  //  auto pNode2 = workSet.lower_bound(Khash); //before

    if(pNode == workSet.end()){
        pNode = workSet.begin();
    }
  //  --pNode2;
    uint32_t Atests,Atestf, Btests,Btestf, R;
    uint32_t A, B;
    A =B = 0;
    Atests = 0;
    Atestf = 0;
    Btests = 0;
    Btestf = 0;
    Atests = pNode->second;//Right child
    Atestf = pNode->first;//left child
    --pNode;
    
    Btests = pNode->second; // Node before second version
    Btestf = pNode->first; //Hash before second version
//std::cout<<" Key " << K<< "Hashed at location " << Khash << "and obtained node for this key is ">> pNode->second<<std::endl; 
// printf(" Key: %u, hashed at location: %u, and obtained node for this key is %u \n", K, Khash, Atests);
//printf(" Node %u with address %u, is before <--Key: %u, and Node: %u, with address %u is after --> the key \n", Btests, Btestf, K, Atests, Atestf);

A = (Atestf - Khash);
 B = (Khash - Btestf);
// printf("Atestf is %u and Btestf is %u and sum is %llu\n", Atestf, Btestf, Atestf+ Btestf);
//printf(" Diff with After is %u and before is %u \n", A, B); 
 
//printf(" Diff with Before is %u \n", B); 
 if (B >= A)  // if B is greater assign to A
 {
    R = Atests; //pNode->second; 
  //  printf(" Key assigned to After Node  \n");
 //   printf(" Key:%u  assigned to After Node: %u with After Node Hash Number is: %u \n", K, Atests, Atestf);
    }
    else  // else A is greater assign to A
    {
    R = Btests; //pNode2->second;
  // printf(" Key assigned to Before Node  \n");
 //  printf(" Key:%u  assigned to Before Node: %u with Before Node Hash Number is: %u \n", K, Btests, Btestf);
}
//printf(" \n \n ");

return R;
//return pNode->second;
}

uint32_t Ring::getSize(){
    return workSet.size() / V;
}


