---
title: java图片处理
date: 2018-08-02 16:34:18
tags: 
- Java
- 图片处理
categories:
- 后端
- Java
copyright: true
---
## 需求
  
将图1~图7拼成图8。  
  
<!--more-->  
  
图1  
  
![](https://i.imgur.com/3DAtxPM.png)
  
图2  
  
![](https://i.imgur.com/RttqkfB.png)
  
图3  
  
![](https://i.imgur.com/rhFHin8.png)
  
图4  
  
![](https://i.imgur.com/R1KkWSO.png)
  
图5  
  
![](https://i.imgur.com/txjpGvB.png)
  
图6  
  
![](https://i.imgur.com/5FFVVNQ.png)
  
图7  
  
![](https://i.imgur.com/40rthiH.png)
  
图8  
  
![](https://i.imgur.com/iGwkG9t.png)
  
## 解决
  
```java
package image;


import com.alibaba.fastjson.JSONObject;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.*;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by huihui on 2018/7/31.
 */
public class GraphicsTest {

    /**
     * 改变图片的大小
     * @param inputStream 上传图片的输入流
     * @param newWidth 新图片的宽
     * @param newHeight 新图片的高
     * @return
     */
    public static BufferedImage resizeImage(InputStream inputStream, int newWidth, int newHeight) {

        //新的图片
        BufferedImage image = new BufferedImage(newWidth, newHeight, BufferedImage.TYPE_INT_RGB);

        try {
            //读取图片
            BufferedImage prevImage = ImageIO.read(inputStream);
            //渲染图片
            Graphics2D graphics2D = image.createGraphics();
            //此类中可以设置图片的透明度
            GraphicsConfiguration graphicsConfiguration = graphics2D.getDeviceConfiguration();
            //设置透明度为原图片的透明度
            image = graphicsConfiguration.createCompatibleImage(newWidth, newHeight, prevImage.getTransparency());
            //释放对象
            graphics2D.dispose();

            //再次渲染图片
            graphics2D = image.createGraphics();
            //创建原图的缩放版本，可以设置图片的缩放算法
            Image imageFrom = prevImage.getScaledInstance(newWidth, newHeight, Image.SCALE_AREA_AVERAGING);
            graphics2D.drawImage(imageFrom, 0, 0, null);
            graphics2D.dispose();

        } catch (IOException e) {
            System.out.println("读取图片失败");
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                System.out.println("输入输出流关闭失败");
            }
        }

        return image;
    }

    /**
     * 在图片上添加水印图片
     * @param bottomImage 底层图片的BufferedImage
     * @param waterImage 水印图片的BufferedImage
     * @param x x坐标
     * @param y y坐标
     * @param alpha 透明度，选择值从0.0到1.0：完全透明到完全不透明
     * @return
     */
    public static BufferedImage waterMark(BufferedImage bottomImage, BufferedImage waterImage, int x, int y, float alpha) {

        Graphics2D graphics2D = bottomImage.createGraphics();
        int waterImageWidth = waterImage.getWidth();
        int waterImageHeight = waterImage.getHeight();
        graphics2D.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, alpha));
        graphics2D.drawImage(waterImage, x, y, waterImageWidth, waterImageHeight, null);
        graphics2D.dispose();

        return bottomImage;
    }

    /**
     * 输出图片
     * @param bufferedImage 要输出图片的BufferedImage
     * @param formatName 要生成图片的类型
     * @param outputStream 生成图片的OutputStream
     */
    public static void generateImage(BufferedImage bufferedImage, String formatName, OutputStream outputStream) {
        try {
            ImageIO.write(bufferedImage, formatName, outputStream);

        } catch (IOException e) {
            System.out.println("图片输出错误");
        }
    }

    public static void main(String[] args) {

        String materialJsonArrayStr = "[\n" +
                "    {\n" +
                "        \"width\": \"1132\",\n" +
                "        \"height\": \"392\",\n" +
                "        \"x\": \"\",\n" +
                "        \"y\": \"\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"width\": \"433\",\n" +
                "        \"height\": \"297\",\n" +
                "        \"x\": \"48\",\n" +
                "        \"y\": \"87\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"width\": \"552\",\n" +
                "        \"height\": \"376\",\n" +
                "        \"x\": \"576\",\n" +
                "        \"y\": \"8\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"width\": \"455\",\n" +
                "        \"height\": \"104\",\n" +
                "        \"x\": \"318\",\n" +
                "        \"y\": \"70\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"width\": \"429\",\n" +
                "        \"height\": \"66\",\n" +
                "        \"x\": \"335\",\n" +
                "        \"y\": \"90\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"width\": \"322\",\n" +
                "        \"height\": \"88\",\n" +
                "        \"x\": \"380\",\n" +
                "        \"y\": \"170\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"width\": \"255\",\n" +
                "        \"height\": \"36\",\n" +
                "        \"x\": \"400\",\n" +
                "        \"y\": \"205\"\n" +
                "    }\n" +
                "]";

        File file1 = new File("src/main/resources/分层_0006_背景.png");
        File file2 = new File("src/main/resources/分层_0000_品1.png");
        File file3 = new File("src/main/resources/分层_0001_品2.png");
        File file4 = new File("src/main/resources/分层_0003_黄色小标签.png");
        File file5 = new File("src/main/resources/分层_0002_每满199减100.png");
        File file6 = new File("src/main/resources/分层_0005_红色小标签.png");
        File file7 = new File("src/main/resources/分层_0004_全球年中购物节.png");
        File presentFile = new File("src/main/resources/成品.png");

        List<File> fileList = new ArrayList<>();
        fileList.add(file1);
        fileList.add(file2);
        fileList.add(file3);
        fileList.add(file4);
        fileList.add(file5);
        fileList.add(file6);
        fileList.add(file7);

        if (!file1.exists() || !file2.exists() || !file3.exists() || !file4.exists() || !file5.exists() || !file6.exists() || !file7.exists()) {
            return;
        }

        List<Material> materialList = JSONObject.parseArray(materialJsonArrayStr, Material.class);

        try {
            for (int i = 0; i < materialList.size(); i++) {
                Material material = materialList.get(i);
                material.setInputStream(new FileInputStream(fileList.get(i)));
                materialList.set(i, material);
            }

            OutputStream outputStream = new FileOutputStream(presentFile);

            String presentImageName = presentFile.getName();
            String formatName = presentImageName.substring(presentImageName.indexOf(".") + 1);

            BufferedImage bottomImage = resizeImage(materialList.get(0).getInputStream(), materialList.get(0).getWidth(), materialList.get(0).getHeight());
            for (int i = 1; i < materialList.size(); i++) {
                BufferedImage waterImage = resizeImage(materialList.get(i).getInputStream(), materialList.get(i).getWidth(), materialList.get(i).getHeight());
                bottomImage = waterMark(bottomImage, waterImage, materialList.get(i).getX(), materialList.get(i).getY(), 1.0f);
            }
            generateImage(bottomImage, formatName, outputStream);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}

class Material {

    private InputStream inputStream;

    private int width;

    private int height;

    private int x;

    private int y;

    public InputStream getInputStream() {
        return inputStream;
    }

    public void setInputStream(InputStream inputStream) {
        this.inputStream = inputStream;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getX() {
        return x;
    }

    public void setX(int x) {
        this.x = x;
    }

    public int getY() {
        return y;
    }

    public void setY(int y) {
        this.y = y;
    }

    @Override
    public String toString() {
        return "Material{" +
                "inputStream=" + inputStream +
                ", width=" + width +
                ", height=" + height +
                ", x=" + x +
                ", y=" + y +
                '}';
    }
}
```
  
## 结果
  
![](https://i.imgur.com/j6SxKRB.png)