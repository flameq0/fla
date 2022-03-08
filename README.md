package com.example.retrfit;
 
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;
import com.alibaba.fastjson.serializer.SerializeConfig;
import com.alibaba.fastjson.serializer.SerializerFeature;
 
import java.io.IOException;
import java.lang.annotation.Annotation;
import java.lang.reflect.Type;
 
import okhttp3.MediaType;
import okhttp3.RequestBody;
import okhttp3.ResponseBody;
import retrofit2.Converter;
import retrofit2.Retrofit;
 
/**
 * @author
 */
public class FastConverterFactory extends Converter.Factory {
    private static final MediaType MEDIA_TYPE = MediaType.parse("application/json; charset=UTF-8");
    private static final Feature[] EMPTY_SERIALIZER_FEATURES = new Feature[0];
 
    private ParserConfig parserConfig = ParserConfig.getGlobalInstance();
    private int featureValues = JSON.DEFAULT_PARSER_FEATURE;
    private Feature[] features;
 
    private SerializeConfig serializeConfig;
    private SerializerFeature[] serializerFeatures;
 
    public FastConverterFactory() {
    }
 
    @Override
    public Converter<ResponseBody, ?> responseBodyConverter(Type type, //
                                                            Annotation[] annotations, //
                                                            Retrofit retrofit) {
        return new ResponseBodyConverter<ResponseBody>(type);
    }
 
    @Override
    public Converter<?, RequestBody> requestBodyConverter(Type type, //
                                                          Annotation[] parameterAnnotations, //
                                                          Annotation[] methodAnnotations, //
                                                          Retrofit retrofit) {
        return new RequestBodyConverter<RequestBody>();
    }
 
    public ParserConfig getParserConfig() {
        return parserConfig;
    }
 
    public FastConverterFactory setParserConfig(ParserConfig config) {
        this.parserConfig = config;
        return this;
    }
 
    public int getParserFeatureValues() {
        return featureValues;
    }
 
    public FastConverterFactory setParserFeatureValues(int featureValues) {
        this.featureValues = featureValues;
        return this;
    }
 
    public Feature[] getParserFeatures() {
        return features;
    }
 
    public FastConverterFactory setParserFeatures(Feature[] features) {
        this.features = features;
        return this;
    }
 
    public SerializeConfig getSerializeConfig() {
        return serializeConfig;
    }
 
    public FastConverterFactory setSerializeConfig(SerializeConfig serializeConfig) {
        this.serializeConfig = serializeConfig;
        return this;
    }
 
    public SerializerFeature[] getSerializerFeatures() {
        return serializerFeatures;
    }
 
    public FastConverterFactory setSerializerFeatures(SerializerFeature[] features) {
        this.serializerFeatures = features;
        return this;
    }
 
    final class ResponseBodyConverter<T> implements Converter<ResponseBody, T> {
        private Type type;
 
        ResponseBodyConverter(Type type) {
            this.type = type;
        }
 
        public T convert(ResponseBody value) throws IOException {
            try {
                return JSON.parseObject(value.string()
                        , type
                        , parserConfig
                        , featureValues
                        , features != null
                                ? features
                                : EMPTY_SERIALIZER_FEATURES
                );
            } finally {
                value.close();
            }
        }
    }
 
    final class RequestBodyConverter<T> implements Converter<T, RequestBody> {
        RequestBodyConverter() {
        }
 
        public RequestBody convert(T value) throws IOException {
            byte[] content = JSON.toJSONBytes(value
                    , serializeConfig == null
                            ? SerializeConfig.globalInstance
                            : serializeConfig
                    , serializerFeatures == null
                            ? SerializerFeature.EMPTY
                            : serializerFeatures
            );
 
            return RequestBody.create(MEDIA_TYPE, content);
        }
    }
}
