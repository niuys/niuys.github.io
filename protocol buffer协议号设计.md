  使用protocol buffer 做通讯协议，有一个问题就是怎么设计协议号，一般会把消息类型放在message序列化的binary之前，或者在message里加一个字段表示message type，这两种办法都需要额外的存储。2.6版本之后支持了oneof，使用它加上pb的反射机制，就可以省略消息类型了，因为这些信息已经隐含在message的字段里了。  
  message BaseMsg {  
    oneof Msg {  
      Msg_1 msg_1 = 1;  
      Msg_2 msg_2 = 2;  
      ...  
    }  
  }  
  
  通过反射接口，可以遍历BaseMsg的字段，因为只会有一个字段是设置的，就可以拿到具体的协议了。
  现在的版本的pb针对oneof生成的c++代码有个问题是，生成的IsInitialized和serialize接口仍然是  
    if(has_msg_1()){  
      ...  
    }  
  if(has_msg_2()){  
    ...  
  }  
   
  在协议非常多的时候，这种代码就有效率问题了，完全可以用已经实现的类似ByteSize的代码  
  switch(Msg_case()){  
    case kmsg_1:{  
      ...  
    }  
    case kmsg_2:{  
      ...  
    }  
  }  
  
  需要修改protoc的实现代码，仿照ByteSize实现起来也不复杂，加上pb的代码质量很高，用了两个多小时就差不多改完了，提了issue，貌似没人回应。
